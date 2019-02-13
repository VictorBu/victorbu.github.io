---
title: Java ���� HLS (m3u8) ��Ƶ
date: 2019-02-02 10:10:00
updated: 2019-02-02 10:10:00
categories: [IT]
tags: [hls,m3u8]
---

# ���������ļ�

```java
public String getIndexFile() throws Exception{
	URL url = new URL(originUrlpath);
	//������Դ
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

# ���������ļ�

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

# ������ƵƬ��

## ͬ������

```java
public HashMap downLoadIndexFile(List<String> urlList){
	HashMap<Integer,String> keyFileMap = new HashMap<>();

	for(int i =0;i<urlList.size();i++){
		String subUrlPath = urlList.get(i);
		String fileOutPath = folderPath + File.separator + i + ".ts";
		keyFileMap.put(i, fileOutPath);
		try{
			downloadNet(preUrlPath + subUrlPath, fileOutPath);

			System.out.println("�ɹ���"+ (i + 1) +"/" + urlList.size());
		}catch (Exception e){
			System.err.println("ʧ�ܣ�"+ (i + 1) +"/" + urlList.size());
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

## ���߳�����

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
			String message = "%s: �߳� " + (startIndex/(endIndex - startIndex) + 1)
					+ ", "+ (i + 1) +"/" + urlList.size() +", �ϼ�: %d";
			try{
				downloadNet(preUrlPath + subUrlPath, fileOutPath);

				System.out.println(String.format(message, "�ɹ�", keyFileMap.size()));
			}catch (Exception e){
				System.err.println(String.format(message, "ʧ��", keyFileMap.size()));
			}
		}
	}
}
```

# ��ƵƬ�κϳ�

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

### ��Դ��ַ��[hlsdownloader](https://github.com/VictorBu/code-snippet/tree/master/java/hlsdownloader)

> �ο���[M3U8������Ƶ�ļ����غϳ�MP4��Ƶ](https://my.oschina.net/haitaohu/blog/1941179)
