---
title: 仿谷歌验证 (Google Authenticator) 的一种 Java 实现
date: 2021-05-28 10:00:00
updated: 2021-05-28 10:00:00
categories: [IT]
tags: [Google Authenticator, Java]
---

Google Authenticator 的原理是服务器随机生成一个密钥并保存并告知客户端。用户需要登陆时客户端根据密钥和时间戳通过一种算法生成一个6位数字的密码。本文使用 java.util.zip.CRC32 模仿 Google Authenticator 实现此功能。

```
    /**
     * 生成验证码
     * @param secret: 密钥
     * @param timeMinute: 时间戳（分钟）
     * @param codeLength: 验证码长度
     * @return 指定长度的数字
     */
    public static String generateCode(String secret, long timeMinute, int codeLength) {
        String key = secret + timeMinute;
        CRC32 crc32 = new CRC32();
        crc32.update(key.getBytes());
        String crc32ValueStr = String.valueOf(crc32.getValue());
        return crc32ValueStr.substring(crc32ValueStr.length() - codeLength); // 可以改成其他规则
    }

    /***
     * 校验验证码
     * @param secret: 密钥
     * @param verifyCode: 待校验验证码
     * @param expireMinute
     * @return
     */
    public static boolean verifyCode(String secret, String verifyCode, int codeLength, int expireMinute) {
        long timeMillisecond = System.currentTimeMillis();
        long timeMinute = minuteByMillisecond(timeMillisecond);

        for (int i = -expireMinute; i <= expireMinute; ++i) {
            String code = generateCode(secret, timeMinute + i, codeLength);
            if(code.equals(verifyCode)) {
                return true;
            }
        }
        return false;
    }

    /***
     * 获取时间戳（分钟）
     * @param timeMillisecond: 时间戳（毫秒）
     * @return
     */
    private static long minuteByMillisecond(long timeMillisecond) {
        return timeMillisecond / (1000 * 60);
    }
```

使用：


```
	final int CODE_LENGTH = 6;
	final int CODE_EXPIRE_MINUTE = 1;
	final String SECRET = "YOUR SECRET";

	long timeMillisecond = System.currentTimeMillis();
	long timeMinute = minuteByMillisecond(timeMillisecond);
	String code = generateCode(SECRET, timeMinute, CODE_LENGTH);
	boolean result = verifyCode(SECRET, code, CODE_LENGTH, CODE_EXPIRE_MINUTE);
```