---
title: Java 下载 HLS (m3u8) 视频
date: 2019-02-02 10:10:00
updated: 2019-02-02 10:10:00
categories: [IT]
tags: [hls,m3u8]
---


# 下载索引文件

```java
public String getIndexFile() throws Exception{
	URL url = new URL(originUrlpath);
	//下载资源
	BufferedReader in = new BufferedReader(new InputStreamReader(url.openStream(),"UTF-8"));

	String content = "" ;
	String line;
	while ((line = in.readLine()) != null) {
		content += line + "\n";
	}
	in.close();

	return content;
}
```

# 解析索引文件

```java
public List analysisIndex(String content) throws Exception{
	Pattern pattern = Pattern.compile(".*ts");
	Matcher ma = pattern.matcher(content);

	List<String> list = new ArrayList<String>();

	while(ma.find()){
		list.add(ma.group());
	}

	return list;
}
```

# 下载视频片段

## 同步下载

```java
public HashMap downLoadIndexFile(List<String> urlList){
	HashMap<Integer,String> keyFileMap = new HashMap<>();

	for(int i =0;i<urlList.size();i++){
		String subUrlPath = urlList.get(i);
		String fileOutPath = folderPath + File.separator + i + ".ts";
		keyFileMap.put(i, fileOutPath);
		try{
			downloadNet(preUrlPath + subUrlPath, fileOutPath);

			System.out.println("成功："+ (i + 1) +"/" + urlList.size());
		}catch (Exception e){
			System.err.println("失败："+ (i + 1) +"/" + urlList.size());
		}
	}

	return  keyFileMap;
}

private void downloadNet(String fullUrlPath, String fileOutPath) throws Exception {

	//int bytesum = 0;
	int byteread = 0;

	URL url = new URL(fullUrlPath);
	URLConnection conn = url.openConnection();
	InputStream inStream = conn.getInputStream();
	FileOutputStream fs = new FileOutputStream(fileOutPath);

	byte[] buffer = new byte[1204];
	while ((byteread = inStream.read(buffer)) != -1) {
		//bytesum += byteread;
		fs.write(buffer, 0, byteread);
	}
}
```

## 多线程下载

```java
public void downLoadIndexFileAsync(List<String> urlList, HashMap<Integer,String> keyFileMap) throws Exception{
	int downloadForEveryThread = (urlList.size() + threadQuantity - 1)/threadQuantity;
	if(downloadForEveryThread == 0) downloadForEveryThread = urlList.size();

	for(int i=0; i<urlList.size();i+=downloadForEveryThread){
		int startIndex = i;
		int endIndex = i + downloadForEveryThread - 1;
		if(endIndex >= urlList.size())
			endIndex = urlList.size() - 1;

		new DownloadThread(urlList, startIndex, endIndex, keyFileMap).start();
	}
}

class DownloadThread extends Thread{
	private List<String> urlList;
	private int startIndex;
	private int endIndex;
	private HashMap<Integer,String> keyFileMap;

	public DownloadThread(List<String> urlList, int startIndex, int endIndex, HashMap<Integer,String> keyFileMap){
		this.urlList = urlList;
		this.startIndex = startIndex;
		this.endIndex = endIndex;
		this.keyFileMap = keyFileMap;
	}
	@Override
	public void run(){
		for(int i=startIndex;i<=endIndex;i++){
			String subUrlPath = urlList.get(i);
			String fileOutPath = folderPath + File.separator + i + ".ts";
			keyFileMap.put(i, fileOutPath);
			String message = "%s: 线程 " + (startIndex/(endIndex - startIndex) + 1)
					+ ", "+ (i + 1) +"/" + urlList.size() +", 合计: %d";
			try{
				downloadNet(preUrlPath + subUrlPath, fileOutPath);

				System.out.println(String.format(message, "成功", keyFileMap.size()));
			}catch (Exception e){
				System.err.println(String.format(message, "失败", keyFileMap.size()));
			}
		}
	}
}
```

# 视频片段合成

```java
public String composeFile(HashMap<Integer,String> keyFileMap) throws Exception{

	if(keyFileMap.isEmpty()) return null;

	String fileOutPath = rootPath + File.separator + fileName;
	FileOutputStream fileOutputStream = new FileOutputStream(new File(fileOutPath));
	byte[] bytes = new byte[1024];
	int length = 0;
	for(int i=0; i<keyFileMap.size(); i++){
		String nodePath = keyFileMap.get(i);
		File file = new File(nodePath);
		if(!file.exists())  continue;

		FileInputStream fis = new FileInputStream(file);
		while ((length = fis.read(bytes)) != -1) {
			fileOutputStream.write(bytes, 0, length);
		}
	}

	return fileName;
}
```

### 开源地址：[hlsdownloader](https://github.com/VictorBu/code-snippet/tree/master/java/hlsdownloader)

> 参考：[M3U8在线视频文件下载合成MP4视频](https://my.oschina.net/haitaohu/blog/1941179)
