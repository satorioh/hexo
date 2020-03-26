---
title: 使用SJCL实现AES GCM加解密
date: 2020-03-26 15:39:03
categories: 加解密
tags:
- sjcl
- aes
permalink: sjcl-aes-gcm-encryption
---
[SJCL项目地址](https://github.com/bitwiseshiftleft/sjcl) [API文档](http://bitwiseshiftleft.github.io/sjcl/doc/) [Demo](http://bitwiseshiftleft.github.io/sjcl/demo/)
<!--more-->

```javascript
SJCL = {
    str2hex(str) {
        if (str === '') return '';
        let arr = [];
        for (let i = 0; i < str.length; i++) {
            arr.push(str.charCodeAt(i).toString(16));
        }
        return arr.join('');
    },

    rightPad(str, targetLength, padChar) {
        return str + Array(targetLength - str.length + 1).join(padChar);
    },

    getKey(pd) {
        if (pd) {
            const pdLen = pd.length;
            if (pdLen > 32) pd = pd.slice(0, 32);
            else if (pdLen > 24 && pdLen < 32) pd = pd.slice(0, 24);
            else if (pdLen > 16 && pdLen < 24) pd = pd.slice(0, 16);
            else if (pdLen < 16) pd = this.rightPad(pd, 16, '0');
        } else {
            pd = this.rightPad('', 32, '0');
        }
        return sjcl.codec.hex.toBits(this.str2hex(pd));
    },

    // 12 bytes base64编码的iv
    getRandomIv() {
        const ivBits = sjcl.random.randomWords(3);
        return sjcl.codec.base64.fromBits(ivBits);
    },

    encrypt(pd, data) {
        const key = this.getKey(pd);
        const iv = this.getRandomIv();
        const encryptedData = sjcl.encrypt(key, JSON.stringify(data), { mode: 'gcm', ts: 128, iv });
        return _.pick(JSON.parse(encryptedData), ['iv', 'ct']);
    },

    decrypt(pd, data) {
        const key = this.getKey(pd);
        const encryptedData = Object.assign(data, { mode: 'gcm', ts: 128 });
        const plainText = sjcl.decrypt(key, JSON.stringify(encryptedData));
        return JSON.parse(plainText);
    },
}
```


