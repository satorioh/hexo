---
title: socket-io demo
date: 2023-07-10 09:51:23
categories: JavaScript
tags:
  - socket.io
permalink: socket-io-demo
---
##### 1.consts.js
```javascript
export const DEFAULT_OPTIONS = { reconnectionAttempts: 10, transports: ["websocket"] };

export const STATUS_CODE = {
  SUCCESS: "SUCCESS",
  RECONNECTING: "RECONNECTING",
  TIMEOUT: "TIMEOUT",
  REPEAT: "REPEAT",
  FAIL: "FAIL"
};

export const STATUS_MSG = {
  SUCCESS: "连接成功",
  RECONNECTING: "连接错误，重连中：",
  REPEAT: "连接已存在，可直接复用",
  TIMEOUT: "连接超时，请检查服务",
  FAIL: "重连失败"
};

```
<!--more-->

##### 2.utils.js
```javascript
import { STATUS_MSG } from "./consts";
import store from "@/store";
import * as types from "@/store/types";

const getStatusMsg = (code, attempt) => {
  if (code === "RECONNECTING") return STATUS_MSG[code] + attempt;
  return STATUS_MSG[code];
};

export const runCallback = (code, socket, callback, attempt) => {
  const msg = getStatusMsg(code, attempt);
  callback({ code, msg, socket });
};

export const findRepeatConnection = (origin) => {
  return store.state.socketConnections.find((item) => item.origin === origin);
};

export const removeFromStore = (origin) => {
  const socketConnections = store.state.socketConnections.filter((item) => item.origin !== origin);
  store.commit(types.SET_SOCKET_CONNECTIONS, socketConnections);
};

```

##### 3.index.js
```javascript
import store from "@/store";
import { io } from "socket.io-client";
import * as types from "@/store/types";
import { DEFAULT_OPTIONS, STATUS_CODE } from "./consts";
import { runCallback, findRepeatConnection, removeFromStore } from "./utils";

export default class SocketConnection {
  constructor(url, options) {
    let origin = new URL(url).origin;
    const repeatConnection = findRepeatConnection(origin);
    if (repeatConnection) {
      Object.assign(this, repeatConnection);
    } else {
      this.origin = origin;
      this.options = Object.assign({}, DEFAULT_OPTIONS, options);
      this.socket = null;
      this.success = false;
      this.timer = null; //链接超时
    }
  }

  link(callback = () => {}) {
    if (this.socket && this.success) {
      runCallback(STATUS_CODE.REPEAT, this, callback);
      return;
    }
    console.log("开始连接");
    if (!this.socket) {
      this.socket = io(this.origin, this.options);
    } else if (this.socket && !this.success) {
      this.socket.connect();
    }

    //链接成功
    this.socket.on("connect", (data) => {
      if (this.timer) clearTimeout(this.timer);
      this.success = true;
      const repeatConnection = findRepeatConnection(this.origin);
      if (!repeatConnection) {
        store.commit(types.SET_SOCKET_CONNECTIONS, [...store.state.socketConnections, this]);
      }
      runCallback(STATUS_CODE.SUCCESS, this, callback);
    });

    //断开链接
    this.socket.on("disconnect", (reason) => {
      this.success = false;
      if (reason === "transport close") console.log("断开链接：连接被关闭(例如:用户失去连接，或者网络由WiFi切换到4G)");
      else if (reason === "io server disconnect")
        console.log("断开链接：服务器使用socket.disconnect()强制断开了套接字。");
      else if (reason === "io client disconnect") console.log("断开链接：使用socket.disconnect()手动断开socket。");
      else if (reason === "ping timeout") console.log("断开链接：服务器没有在pingInterval + pingTimeout范围内发送PING");
      else if (reason === "transport error")
        console.log("断开链接：连接遇到错误(例如:服务器在HTTP长轮询周期期间被杀死)");
      else console.log("断开链接：", reason);
    });

    this.socket.io.on("reconnect_attempt", (attempt) => {
      const msg = `连接错误，重连中：${attempt}`;
      console.log(msg);
      runCallback(STATUS_CODE.RECONNECTING, this, callback, attempt);
    });

    this.socket.io.on("reconnect_failed", () => {
      const msg = `重连失败`;
      console.log(msg);
      runCallback(STATUS_CODE.FAIL, this, callback);
      this.closeLink();
    });

    // 首次连接超时处理（5s）
    this.timer = setTimeout(() => {
      if (!this.success) {
        this.closeLink();
        runCallback(STATUS_CODE.TIMEOUT, this, callback);
      }
    }, 5000);
  }

  closeLink() {
    this.socket.close();
    this.socket = null;
    removeFromStore(this.origin);
  }

  listen(eventName, callback) {
    if (this.socket._callbacks[`$${eventName}`]) return;
    this.socket.on(eventName, callback);
  }

  send(eventName, data) {
    return new Promise((resolve, reject) => {
      this.socket.emit(eventName, data, (ack) => {
        resolve(ack);
      });
    });
  }
}

```

##### reference
```javascript
import { io } from "socket.io-client";
import Methods from "./methods.js";
import store from "@/store";
import * as types from "@/store/types";

//socket建立链接
export const linkSocket = ({ url, options, eventName, callback = () => {} }) => {
  let obj = {
    url: url,
    socket: null,
    success: false // 是否链接成功
  };

  let timer = null; //链接超时

  //这里是避免多个socket链接
  store.state.socketArray.forEach((item) => {
    if (url === item.url) {
      obj = item;
      if (item.success) {
        if (timer) clearTimeout(timer); //清空超时
        callback({ code: 1, data: "已连接" });
      } else {
        setTimeout(() => {
          linkSocket({ url, options, eventName, callback });
        }, 1000);
      }
    }
  });

  if (obj.socket && !obj.success) return;

  if (!obj.socket) {
    obj.socket = io(url, options);
    store.commit(types.SET_SOCKETARRAY, [...store.state.socketArray, obj]);
  }
  //链接成功
  obj.socket.on("connect", (data) => {
    if (timer) clearTimeout(timer);
    obj.success = true;
    callback({ code: 1, data: "连接成功" });
    console.log(obj.socket);
    socketOn(eventName, obj.socket); // 开启监听
  });

  //断开链接
  obj.socket.on("disconnect", (reason) => {
    obj.success = false;
    if (reason === "transport close") console.log("断开链接：连接被关闭(例如:用户失去连接，或者网络由WiFi切换到4G)");
    else if (reason === "io server disconnect")
      console.log("断开链接：服务器使用socket.disconnect()强制断开了套接字。");
    else if (reason === "io client disconnect") console.log("断开链接：使用socket.disconnect()手动断开socket。");
    else if (reason === "ping timeout") console.log("断开链接：服务器没有在pingInterval + pingTimeout范围内发送PING");
    else if (reason === "transport error") console.log("断开链接：连接遇到错误(例如:服务器在HTTP长轮询周期期间被杀死)");
    else console.log("断开链接：", reason);
  });

  obj.socket.on("connect_error", (data) => {
    console.log(`连接错误，重连中：${data}`);
  });

  // 首次连接超时处理（5s）
  timer = setTimeout(() => {
    if (!obj.success) {
      obj.socket.close();
      const socketArray = store.state.socketArray.filter((item) => {
        return item.url !== obj.url;
      });
      store.commit(types.SET_SOCKETARRAY, socketArray);
      callback({ code: 0, data: "连接超时，检查服务" });
    }
  }, 5000);
};

export const socketOn = async (eventName, socket) => {
  //是否存在监听，避免重复监听
  if (socket._callbacks[`$${eventName}`]) return;
  socket.on(eventName, async (data, callback) => {
    //执行对应的方法
    if (Methods[eventName]) Methods[eventName](data, callback);
  });
};

/**
 * @eventName 发送方法名
 * @data 发送内容
 */
export const socketEmit = async (eventName, data) => {
  // eslint-disable-next-line no-async-promise-executor
  return new Promise(async (resolve) => {
    const res = await getSocket(data);
    if (res.code === 0) {
      resolve(res);
    } else {
      res.socket.emit(eventName, data);
      resolve({ code: 1, data: "发送成功" });
    }
  });
};

//获取 socket 对象
export const getSocket = (data) => {
  let obj = { url: data.url, socket: null, success: false }; //success:是否链接成功

  store.state.socketArray.forEach((item) => {
    if (data.url === item.url) obj = item;
  });
  if (!obj.socket) return { code: 0, data: "没查到socket连接" };
  if (!obj.success) return { code: 0, data: "socket未正常连接" };
  return { code: 1, socket: obj.socket };
};

```
