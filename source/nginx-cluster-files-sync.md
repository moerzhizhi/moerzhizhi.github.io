---
title: 【转】Nginx负载均衡之后的文件上传同步
date: 2018-10-31 23:08:00 +0800
updated: 2018-10-31 23:08:00 +0800
comments: false
---
使用 Nginx进行负载均衡后（使用 ip_hash; 策略），存在一个问题，有 A、B 两台服务器，使用相同的数据库，用户访问被分配到了 A 服务器，成功上传一张照片，刷新后在 A 服务器可以看到；但是连接 B 服务器的用户刷新页面后只看到了一条记录，照片却丢失显示不了；网上有比较多的解决办法，但是都集中在使用 WADI 等配置文件同步策略，看过后觉得有点复杂（WADI 本身配置比较简单），想简单点实现，思路如下：预先在配置文件中写好参与负载均衡的服务器列表，用户在 A 上提交照片后，就检测系统中配置的集群服务器的列表，排除 A 后通过 linux 的 scp 命令在不同的服务器之间实现文件同步，就一个简单的 Java 类就可以实现了，实现 linux 同步是采用的 ganymed-ssh2 库包提供的 ssh 的功能。
```Java
package zkyg.util;

import ch.ethz.ssh2.Connection;
import ch.ethz.ssh2.SCPClient;

import java.io.IOException;
import java.net.InetAddress;
import java.net.UnknownHostException;
import java.util.ArrayList;
import java.util.ResourceBundle;

import zkyg.util.Log;

/**
 * 解决负载均衡情况下的文件上传不一致的情况：一旦用户上传照片到A服务器，上传完毕后调用Linux的scp命令来实现不同的服务器之间的文件同步操作：
 * 
 * @author wangyq
 * 
 */
public class ClusterFilesSync extends Thread {
	private static String remoteUser = "root";
	private static String remotePass = "randompsw@123";
	private String absFileName = "";

	public ClusterFilesSync(String absFileName) {
		this.absFileName = absFileName;
	}

	@Override
	public void run() {
		try {
			ArrayList<String> al = getHosts();
			for (String s : al) {
				// 循环同步每台服务器上的文件
				sync(s, 22, absFileName);
			}
		} catch (Exception e) {
			e.printStackTrace();
		}

	}

	/**
	 * 获取需要同步的服务器列表
	 * 
	 * @return
	 */
	public static ArrayList<String> getHosts() {
		// 在com.firm.config目录下有system.properties文件记录了集群服务器列表
		ResourceBundle bd = ResourceBundle.getBundle("com.firm.config.system");
		String clusters = bd.getString("clusters");
		String[] ips = clusters.split(",");
		ArrayList<String> al = new ArrayList<String>();
		for (String s : ips) {
			al.add(s);
		}
		String localIp = "";
		try {
			// 获取当前操作的服务器的IP地址，linux的etc/hosts中要配置好网络地址，不然只会得到127.0.0.1
			localIp = InetAddress.getLocalHost().getHostAddress();
			System.out.println("本机IP地址为：" + localIp);
		} catch (UnknownHostException e) {
			e.printStackTrace();
		}
		al.remove(localIp);
		return al;
	}

	/**
	 * 简单的文件同步
	 * 
	 * @return
	 */
	public static void sync(String ip, int port, String absFileName) {
		String path = "/var/www/html/jetty/webapps/userphotos/";
		Connection con = new Connection(ip, port);
		// 连接
		try {
			con.connect();
			// 远程服务器的用户名密码
			boolean isAuthed = con.authenticateWithPassword(remoteUser, remotePass);
			// 建立SCP客户端
			if (isAuthed) {
				SCPClient scpClient = con.createSCPClient();
				// 将文件同步到指定的path目录下面
				scpClient.put(absFileName, path);
				Log.console("文件同步成功");
			} else {
				Log.console("文件同步失败");
			}
			con.close();
		} catch (IOException e) {
			e.printStackTrace();
		}
	}

}
```
在用户图片上传成功后进行调用。
```Java
if (true) {
	// 集群模式下进行数据同步
	new zkyg.util.ClusterFilesSync(fileName).start();
}
```

[原文链接](https://blog.csdn.net/educast/article/details/79739160)