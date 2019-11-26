```
import axios from 'axios';
import iView from 'view-design';
import store from '@/store';
// import { Spin } from 'view-design'
const CancelToken = axios.CancelToken;
class HttpRequest {
    constructor(baseUrl = baseURL) {
        this.baseUrl = baseUrl;
        this.queue = {};
        this.cancels = {};
    }
    getInsideConfig() {
        const config = {
            baseURL: this.baseUrl,
            timeout: 30000,
            headers: {
                //
            }
        };
        return config;
    }
    destroy(url) {
        delete this.queue[url];
        if (!Object.keys(this.queue).length) {
            // Spin.hide()
        }
    }
    interceptors(instance, url) {
        // 请求拦截
        instance.interceptors.request.use(
            config => {
                // 添加全局的loading...
                if (!Object.keys(this.queue).length) {
                    // Spin.show() // 不建议开启，因为界面不友好
                }
                this.queue[url] = true;
                return config;
            },
            error => {
                return Promise.reject(error);
            }
        );
        // 响应拦截
        instance.interceptors.response.use(
            res => {
                this.destroy(url);
                delete this.cancels[url];
                const { data, status } = res;
                if (!data.success && data.code !== '10000') {
                    iView.Message.error(data.message || `错误.code:${data.code}`);
                }
                if (data.code === '10000') {
                    // 是否是获取用户信息/状态的接口
                    // TODO：如果获取用户状态接口无登陆验证则不需该校验
                    const isGetInfo = ~res.config.url.indexOf('users/cur-user');
                    if (!isGetInfo) {
                        store.dispatch('handleLogOut');
                    }
                }
                return { data, status };
            },
            error => {
                this.destroy(url);
                if (error.code === 'ECONNABORTED' && error.message.indexOf('timeout') !== -1) {
                    // console.log('根据你设置的timeout/真的请求超时 判断请求现在超时了，你可以在这里加入超时的处理方案')
                    iView.Message.warning({
                        content: '请求超时',
                        duration: 10,
                        closable: true
                    });
                }
                if (error && error.response) {
                    switch (error.response.status) {
                        case 400:
                            error.message = '请求错误';
                            break;

                        case 401:
                            error.message = '未授权，请登录';
                            break;

                        case 403:
                            error.message = '拒绝访问';
                            break;

                        case 404:
                            error.message = '请求地址出错';
                            break;

                        case 408:
                            error.message = '请求超时';
                            break;
                        case 413:
                            error.message = '上传文件过大！';
                            break;
                        case 500:
                            error.message = '服务器内部错误';
                            break;

                        case 501:
                            error.message = '服务未实现';
                            break;

                        case 502:
                            error.message = '网关错误';
                            break;

                        case 503:
                            error.message = '服务不可用';
                            break;

                        case 504:
                            error.message = '网关超时';
                            break;

                        case 505:
                            error.message = 'HTTP版本不受支持';
                            break;

                        default:
                    }
                }
                iView.Message.warning({
                    content: error.message || '服务出错了',
                    duration: 1000,
                    closable: true
                });
                return Promise.reject(error);
            }
        );
    }
    request(options) {
        const instance = axios.create();
        options = Object.assign(this.getInsideConfig(), options);
        if (this.cancels[options.url]) {
            this.cancels[options.url].forEach(c => {
                c('cancel');
            });
        }
        options.cancelToken = new CancelToken(c => {
            this.cancels[options.url] = [c];
        });
        if (options.method === 'get' || options.method === undefined) {
            if (options.params) {
                options.params._t = new Date().getTime();
            }
            else {
                options.params = {
                    _t: new Date().getTime()
                };
            }
        }
        this.interceptors(instance, options.url);
        return instance(options);
    }
}
export default HttpRequest;

```
