---
title: 扫描二维码（登录，支付）后立即通知
date: 2018-03-05 10:48:42
tags:
top: 1000
---

![](http://www.wailian.work/images/2018/03/05/1211.jpg)

> 最近在做微信的扫码支付，遇到一个问题：如何在用户扫码支付完成之后，客户端立即得到通知，进行下一步的跳转？

首先想到的策略可能是客户端轮询查询订单状态，根据返回结果进行跳转

这个方式有明显的缺点，轮询时间设置短，频繁发送请求，对服务器以及数据库都会产生压力；轮询时间过长，用户等待时间长，体验很差；

针对这个问题想到了微信网页版的扫码登录（扫码完成后，立即登录），现在研究一下它的原理并实现相同的功能

# 微信扫码登录原理

![pengding](http://www.wailian.work/images/2018/03/05/pending.png)

根据图片中，前端二维码页面发送一个网络请求，但是这个请求并没有立即返回

![408](http://www.wailian.work/images/2018/03/05/408.png)

一段时间没有扫描后，后端返回408，前端重新发起一个相同的网络请求，并继续pending

据此猜测大概实现原理如下：

1. 进入网站-生成一个唯一标识(比如UUID)
2. 跳转到二维码页面（二维码中的链接包含次UUID）
3. 二维码页面向服务端发起请求，查询二维码是被扫登录
4. 服务器收到请求，查询。如果未扫登录，进入等待(wait)，不立即返回
5. 一旦被扫，立即返回(notify)
6. 页面收到结果，做后续处理

步骤大概就是如此，但是有个问题，步骤3如果请求超时，如何处理？处理方式是，一段固定时间后，返回408（timeout）

# UUID缓存

```
public static Map<String, ScanPool> cacheMap = new ConcurrentHashMap<String, ScanPool>();
```

一定要使用ConcurrentHashMap否则多线程操作集合会报错ConcurrentModificationException

单线程中出现该异常的原因是，对一个集合遍历的同时，又对该集合进行了增删的操作

多线程中更易出现该异常，当你在一个线程中对一数据集合进行遍历，正赶上另外一个线程对该数据集合进行增删操作时便会出现该异常

缓存还要设置自动清理功能，防止增长过大

# 生成二维码

```
@RequestMapping("/qrcode/{uuid}")
@ResponseBody
String createQRCode(@PathVariable String uuid, HttpServletResponse response) {
    System.out.println("生成二维码");

    String text = "http://2b082e46.ngrok.io/login/" + uuid;
    int width = 300;
    int height = 300;
    String format = "png";
    //将UUID放入缓存
    ScanPool pool = new ScanPool();
    PoolCache.cacheMap.put(uuid, pool);
    try {
        Map<EncodeHintType, Object> hints = new HashMap<EncodeHintType, Object>();
        hints.put(EncodeHintType.CHARACTER_SET, "utf-8");
        hints.put(EncodeHintType.ERROR_CORRECTION, ErrorCorrectionLevel.H); //容错率
        BitMatrix bitMatrix = new MultiFormatWriter().encode(text, BarcodeFormat.QR_CODE, width, height, hints);
        MatrixToImageWriter.writeToStream(bitMatrix, format, response.getOutputStream());
    } catch (WriterException e) {
        e.printStackTrace();
    } catch (IOException e) {
        e.printStackTrace();
    }
    return null;
}
```

生成二维码，并将UUID放入缓存中

此处需要注意，二维码url必须是外网可以访问地址，此处可以使用[内网穿透工具](https://ngrok.com/)


# 验证是否登录

前端发起请求，验证该二维码是否已经被扫登录

```
@RequestMapping("/pool")
@ResponseBody
String pool(String uuid) {
    System.out.println("检测[" + uuid + "]是否登录");

    ScanPool pool = PoolCache.cacheMap.get(uuid);

    if (pool == null) {
        return "timeout";
    }

    //使用计时器，固定时间后不再等待扫描结果--防止页面访问超时
    new Thread(new ScanCounter(pool)).start();

    boolean scanFlag = pool.getScanStatus();

    if (scanFlag) {
        return "success";
    } else {
        return "fail";
    }
}
```

获得状态

```
public synchronized boolean getScanStatus() {
    try {
        if (!isScan()) { //如果还未扫描，则等待
            this.wait();
        }
        if (isScan()) {
            return true;
        }
    } catch (InterruptedException e) {
        // TODO Auto-generated catch block
        e.printStackTrace();
    }
    return false;
}

public synchronized void notifyPool() {
    try {
        this.notifyAll();
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```

新开线程防止页面访问超时

```
class ScanCounter implements Runnable {

    public Long timeout = 27000L;

    //传入的对象
    private ScanPool scanPool;

    public ScanCounter(ScanPool scanPool) {
        this.scanPool = scanPool;
    }

    @Override
    public void run() {
        try {
            Thread.sleep(timeout);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        notifyPool(scanPool);
    }

    public synchronized void notifyPool(ScanPool scanPool) {
        scanPool.notifyPool();
    }
}
```

![verify](http://www.wailian.work/images/2018/03/05/code4c40c.png)

# 定时清理uuid

为防止cacheMap不断增加的问题，需要在静态代码块中开启线程定时清理

```
public class PoolCache {
    //缓存超时时间 80秒
    private static Long timeOutSecond = 80L;

    //每1分钟清理一次缓存
    private static Long cleanIntervalSecond = 60L;

    public static Map<String, ScanPool> cacheMap = new ConcurrentHashMap<String, ScanPool>();

    static {
        new Thread(new Runnable() {

            @Override
            public void run() {
                while (true) {
                    try {
                        Thread.sleep(cleanIntervalSecond * 1000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    clean();
                }
            }

            public void clean() {
                System.out.println("缓存清理...");

                if (cacheMap.keySet().size() > 0) {
                    Iterator<String> iterator = cacheMap.keySet().iterator();
                    while (iterator.hasNext()) {
                        String key = iterator.next();
                        ScanPool pool = cacheMap.get(key);
                        if (System.currentTimeMillis() - pool.getCreateTime() > timeOutSecond * 1000) {
                            cacheMap.remove(key);
                            // 这一行很关键！用于当清理完成，前端请求还在pending时，立即返回结果
                            pool.notifyPool();
                        }
                    }
                }
            }
        }).start();
    }

}
```


# 扫码

```
@RequestMapping("/login/{uuid}")
@ResponseBody
String login(@PathVariable String uuid) {

    ScanPool pool = PoolCache.cacheMap.get(uuid);

    if (pool == null) {
        return "timeout,scan fail";
    }
    
    // 设置被扫状态，唤起线程
    pool.scanSuccess();

    return "扫码完成，登录成功";
}
```

扫码成功，设置扫码状态，唤起线程

```
public synchronized void scanSuccess() {
    try {
        setScan(true);
        this.notifyAll();
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```

![](http://www.wailian.work/images/2018/03/05/ok.png)

手机扫码后

![](http://www.wailian.work/images/2018/03/05/mobile.jpg)


对比[完整代码](https://github.com/wangweiye01/scan_login)很容易看实现原理
