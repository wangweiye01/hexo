---
title: socket实现服务端客户端通讯
date: 2017-12-03 13:00:29
tags:
---
# server服务端

## 主线程

### 构建页面
### 创建serverSocket
### 添加send按钮的点击事件

## 子线程

### 死循环接收消息

## 服务端代码如下:

```
/**
 * 主线程发送消息
 * 子线程接收消息
 */
package com.yatai.web;

import java.awt.BorderLayout;
import java.awt.Container;
import java.awt.FlowLayout;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
import java.io.DataInputStream;
import java.io.DataOutputStream;
import java.io.IOException;
import java.net.ServerSocket;
import java.net.Socket;
import java.util.ArrayList;

import javax.swing.JButton;
import javax.swing.JFrame;
import javax.swing.JPanel;
import javax.swing.JScrollPane;
import javax.swing.JTextArea;
import javax.swing.JTextField;

/**
 * @author 王伟业 2014年5月28日
 */
public class ChatServer implements ActionListener, Runnable {
    // 用来存放客户端连接数量
    public static ArrayList<Socket> socketList = new ArrayList<Socket>();
    // 显示历史聊天记录
    private JTextArea showArea;
    // 待发送的字符区域
    private JTextField msgText;
    // 窗口
    private JFrame mainJframe;
    // 发送按钮
    private JButton sentBtn;
    // 滚动面板
    private JScrollPane JSPane;
    // 普通面板
    private JPanel pane;
    // 最大的容器
    private Container con;
    // 线程处理信息
    private Thread thread = null;
    private ServerSocket serverSocket;
    private Socket connectToClient;
    private DataInputStream inFromClient;
    private DataOutputStream outToClient;

    /**
     * 构造函数用来设置界面，处理事件
     */
    public ChatServer() {
        // TODO Auto-generated constructor stub
        // 设置页面
        mainJframe = new JFrame("服务器端");
        // 初始化容器
        con = mainJframe.getContentPane();
        showArea = new JTextArea();
        showArea.setEditable(false); // 历史聊天窗口中的文字域不能编辑，只供查看
        showArea.setLineWrap(true); // 自动换行
        JSPane = new JScrollPane(showArea);
        // 待发送文字区域
        msgText = new JTextField();
        msgText.setColumns(35);
        // 事件监听
        msgText.addActionListener(this);
        sentBtn = new JButton("Send");
        sentBtn.addActionListener(this);
        // 界面下部
        pane = new JPanel();
        pane.setLayout(new FlowLayout());
        pane.add(msgText);
        pane.add(sentBtn);

        con.add(JSPane, BorderLayout.CENTER);
        con.add(pane, BorderLayout.SOUTH);
        mainJframe.setSize(500, 400);
        mainJframe.setLocation(600, 200);
        mainJframe.setVisible(true);
        mainJframe.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

        try {
            serverSocket = new ServerSocket(8888);
            showArea.append("  正在等待对话请求\n");
            // 监听端口
            connectToClient = serverSocket.accept();
            inFromClient = new DataInputStream(connectToClient.getInputStream());
            outToClient = new DataOutputStream(connectToClient.getOutputStream());
            // 启动线程
            thread = new Thread(this);
            thread.setPriority(Thread.MIN_PRIORITY);
            thread.start();
        } catch (IOException e) {
            // TODO Auto-generated catch block
            // 出现此异常说明服务器未创建成功
            showArea.append("  对不起，不能创建服务器\n");
            // 待发送文字区域不能编辑
            msgText.setEditable(false);
            // 发送按钮不可用
            sentBtn.setEnabled(false);
        }

    }

    /*
     * (non-Javadoc)
     * 
     * @see java.lang.Runnable#run()
     */
    @Override
        public void run() {
            // TODO Auto-generated method stub
            // 此线程用来接收客户端传来的信息
            while (true) {
                try {
                    showArea.append("  对方说：" + inFromClient.readUTF() + "\n");
                    Thread.sleep(1000);
                } catch (IOException | InterruptedException e) {
                    // TODO Auto-generated catch block
                    e.printStackTrace();
                }
            }

        }

    /*
     * (non-Javadoc)
     * 
     * @see
     * java.awt.event.ActionListener#actionPerformed(java.awt.event.ActionEvent)
     */
    @Override
        public void actionPerformed(ActionEvent e) {
            // TODO Auto-generated method stub
            // 响应按钮事件
            String s = msgText.getText();
            // 如果待发送文字区域存在文字
            if (s.length() > 0) {
                try {
                    // 将文字写入到流中
                    outToClient.writeUTF(s);
                    outToClient.flush();
                    // 历史聊天记录增添内容
                    showArea.append("  我说：" + msgText.getText() + "\n");
                    // 待发送文字区域设为空
                    msgText.setText(null);
                } catch (IOException e1) {
                    // TODO Auto-generated catch block
                    // 出现此异常说明消息未发送成功
                    showArea.append("  你的消息：" + "“" + msgText.getText() + "”" + "未能发送成功\n");
                }
            }
        }

    public static void main(String[] args) {
        // 主线程用来发送消息
        new ChatServer();
    }

}
```

# client客户端

## 主线程

### 构建页面
### 连接serverSocket
### 添加send按钮的监听事件

## 子线程

### 死循环接收消息

### 客户端代码如下：
```
/**
 * 
 */
package com.yatai.web;

import java.awt.BorderLayout;
import java.awt.Container;
import java.awt.FlowLayout;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
import java.io.DataInputStream;
import java.io.DataOutputStream;
import java.io.IOException;
import java.net.Socket;

import javax.swing.JButton;
import javax.swing.JFrame;
import javax.swing.JPanel;
import javax.swing.JScrollPane;
import javax.swing.JTextArea;
import javax.swing.JTextField;

/**
 * @author 王伟业 2014年5月28日
 */
public class ChatClient implements ActionListener, Runnable {
    // 相同的界面形式
    private JTextArea showArea;
    private JTextField msgText;
    private JFrame mainJframe;
    private JButton sentBtn;
    private JScrollPane JSPane;
    private JPanel pane;
    private Container con;
    // 相似的处理方法
    private Thread thread = null;
    private Socket connectToServer;
    private DataInputStream inFromServer;
    private DataOutputStream outToServer;

    /**
     * 
     */
    public ChatClient() {
        // TODO Auto-generated constructor stub
        // 构造函数下完成以下内容
        mainJframe = new JFrame("客户端");
        con = mainJframe.getContentPane();
        showArea = new JTextArea();
        showArea.setEditable(false);
        showArea.setLineWrap(true);
        showArea.setWrapStyleWord(true);
        JSPane = new JScrollPane(showArea);
        msgText = new JTextField();
        msgText.setColumns(35);
        msgText.addActionListener(this);
        sentBtn = new JButton("Send");
        sentBtn.addActionListener(this);
        pane = new JPanel();
        pane.setLayout(new FlowLayout());
        pane.add(msgText);
        pane.add(sentBtn);
        con.add(JSPane, BorderLayout.CENTER);
        con.add(pane, BorderLayout.SOUTH);
        mainJframe.setSize(500, 400);
        mainJframe.setLocation(80, 200);
        mainJframe.setVisible(true);
        mainJframe.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

        try {
            connectToServer = new Socket("localhost", 8888);
            inFromServer = new DataInputStream(connectToServer.getInputStream());
            outToServer = new DataOutputStream(connectToServer.getOutputStream());
            showArea.append("  连接成功，可以通信\n");

            // 创建线程
            thread = new Thread(this);
            thread.setPriority(Thread.MIN_PRIORITY);
            thread.start();
        } catch (IOException e) {
            // TODO Auto-generated catch block
            // 出现异常说明连接失败
            // 向历史聊天区域打印提示信息
            showArea.append("  对不起，连接服务器失败\n");
            // 异常连接时输入框不可用
            msgText.setEditable(false);
            msgText.setEnabled(false);
        }
    }

    /*
     * (non-Javadoc)
     * 
     * @see java.lang.Runnable#run()
     */
    @Override
        public void run() {
            // TODO Auto-generated method stub
            // 该线程用来接收传来的消息
            while (true) {
                try {
                    showArea.append("  对方说：" + inFromServer.readUTF() + "\n");
                    Thread.sleep(1000);
                } catch (IOException | InterruptedException e) {
                    // TODO Auto-generated catch block
                    // 此处异常处理。。。。
                    e.printStackTrace();
                }
            }
        }

    /*
     * (non-Javadoc)
     * 
     * @see
     * java.awt.event.ActionListener#actionPerformed(java.awt.event.ActionEvent)
     */
    @Override
        public void actionPerformed(ActionEvent e) {
            // TODO Auto-generated method stub
            // 响应事件
            String s = msgText.getText();
            // 如果待发送文字区域存在文字
            if (s.length() > 0) {
                try {
                    // 将文字写入到流中
                    outToServer.writeUTF(s);
                    outToServer.flush();
                    // 历史聊天记录增添内容
                    showArea.append("  我说：" + msgText.getText() + "\n");
                    // 待发送文字区域设为空
                    msgText.setText(null);
                } catch (IOException e1) {
                    // TODO Auto-generated catch block
                    // 出现此异常说明消息未发送成功
                    showArea.append("  你的消息：" + "“" + msgText.getText() + "”" + "未能发送成功\n");
                }
            }
        }

    public static void main(String[] args) {
        new ChatClient();
    }
}
```
