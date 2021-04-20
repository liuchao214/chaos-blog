---
title: Http请求终止与拦截方法记录

date: 2020-08-19 20:51:46

index_img: /images/solution.png

tags: [webpack, Vue, JavaScript]
categories: [工作杂记]
---
前言
------
> 基于axios的http请求响应拦截与请求拦截，可自动终止重复请求，也提供手动终止方法

代码
------

```js
import axios from 'axios';
import Cookies from 'js-cookie';
import {clearLoginInfo} from '@/utils';
import qs from 'qs';
import isPlainObject from 'lodash/isPlainObject';

const http = axios.create({
    timeout: 1000 * 180,
    withCredentials: true
});

/**
 * 添加终止请求方法
 * @param {Object} config
 */
http.cancelPending = (config) => {
    if (http.pendingRequests
        && http.pendingRequests[`${config.url}&${config.method}`]) {
        http.pendingRequests[`${config.url}&${config.method}`]('canceled'); // 执行取消操作
        delete http.pendingRequests[`${config.url}&${config.method}`];
    }
};

/**
 * 请求拦截
 */
http.interceptors.request.use(
    (config) => {
        let noCancel = false;
        // token
        Object.defineProperty(config, 'headers', {
            value: {
                ...config.headers,
                access_token: Cookies.get('token')
            }
        });
        if (config.method === 'get') {
            noCancel = config.params?.noCancel;
            if (noCancel != null) {
                delete config.params.noCancel;
            }
            Object.defineProperty(config, 'params', {
                value: {
                    ...config.params,
                    unique_t: new Date().getTime()
                }
            });
        }
        if (isPlainObject(config.data)) {
            noCancel = config.data.noCancel;
            if (noCancel != null) {
                delete config.data.noCancel;
            }
            Object.defineProperty(config, 'data', {
                value: {
                    ...config.data
                }
            });
            if (/^application\/x-www-form-urlencoded/.test(
                config.headers['content-type'])) {
                Object.defineProperty(config, 'data', {
                    value: qs.stringify(config.data)
                });
            }
        }
        if (!noCancel && http.cancelPending) {
            http.cancelPending(config); // 如果没有配置阻止取消请求（noCancel），在一个请求发送前执行取消操作
            Object.defineProperty(config, 'cancelToken', {
                value: new axios.CancelToken((cancelFn) => {
                    if (http.pendingRequests) {
                        http.pendingRequests[`${config.url}&${config.method}`] = cancelFn;
                    } else {
                        http.pendingRequests = {};
                        http.pendingRequests[`${config.url}&${config.method}`] = cancelFn;
                    }
                }),
                writable: false
            });
        }
        return config;
    },
    (error) => {
        return Promise.reject(error);
    }
);

/**
 * 响应拦截
 */
http.interceptors.response.use(
    (response) => {
        if (http.cancelPending) {
            http.cancelPending(response.config);
        }
        return response.data;
    },
    (error) => {
        if (error?.response?.status === 401) {
            clearLoginInfo();
        }
        return Promise.reject(error);
    }
);

export default http;
```
