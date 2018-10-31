---
title: Java 取得当前程序部署的服务器 IP
comments: false
---
## 问题
之前使用 `InetAddress.getLocalHost().getHostAddress()` 时，在开发机上可以取得正确的 IP，但是部署到 Linux 服务器之后，取到的值变成了 `127.0.0.1` 。如何能取到真正的 IP 呢？
## 方案
``` java
public static InetAddress getCurrentIp() {
	try {
		Enumeration<NetworkInterface> networkInterfaces = NetworkInterface.getNetworkInterfaces();
		while (networkInterfaces.hasMoreElements()) {
			NetworkInterface ni = (NetworkInterface) networkInterfaces.nextElement();
			Enumeration<InetAddress> nias = ni.getInetAddresses();
			while (nias.hasMoreElements()) {
				InetAddress ia = (InetAddress) nias.nextElement();
				if (!ia.isLinkLocalAddress() && !ia.isLoopbackAddress() && ia instanceof Inet4Address) {
					return ia;
				}
			}
		}
	} catch (SocketException e) {
		e.printStackTrace();
	}
	return null;
}
```