# SEP
SEP是一个简单的JS服务端框架

SEP框架将于稍后基础方法完善后开源，你可以搭配SRP实现更丰富的功能。

## 代码片段

以下SEP代码片段部分（经允许）参考于，几位朋友的已开源，或即将开源的项目。
同时已被多位周边studio开发伙伴请求分享，部分片段已被应用于他们的开源项目中。  
为了方便更多朋友参考，所以将这部分片段放在下方：

```
function getParser() {
    return function (req, res, next) {
        req.GET = querystring.parse(req.url.substr(req.url.indexOf('?') + 1));
        next();
    };
}
function cookieParser() {
    return function (req, res, next) {
        req.COOKIE = querystring.parse(req.headers.cookie);
        next();
    };
}
Buffer.prototype.split=Buffer.prototype.split || function (spl) {//spl为分隔符
    let arr=[];//分隔出来内容
    let cur=0;//当前遍历位置
    let n=0;//索引到的位置
    while((n=this.indexOf(spl,cur))!=-1){
    //如果索引值存在（不为-1）, 将索引到位置给n。
        arr.push(this.slice(cur,n));//切割，并存到数组
        cur=n+spl.length;//向后遍历，寻找下一分隔符
    }
    arr.push(this.slice(cur));
    return arr;
}
function bodyParser(upload) {
    let uid = 0;
    return async function (req, res, next) {
        uid++;
        let id = 0;
        let arr = [];
        req.on('data', function (data) {
            arr.push(data);
        });
        req.on('end', function () {
            let data = Buffer.concat(arr);
            let post = {};
            if (req.method == 'POST') {
                if (req.headers['content-type'].indexOf('application/x-www-form-urlencoded') != -1) {
                    post = querystring.parse(data.toString());
                } else if (req.headers['content-type'].indexOf('multipart/form-data') != -1) {
                    let str = req.headers['content-type'].split(';')[1];
                    if (str) {
                        let boundary = '--' + str.split('=')[1];
                        let array = data.split(boundary);
                        array.shift();
                        array.pop();
                        array = array.map((elem) => elem.slice(2, elem.length - 2));
                        array.forEach(elem => {
                            let n = elem.indexOf('\r\n\r\n');
                            let disposition = elem.slice(0, n);
                            let content = elem.slice(n + 4);
                            disposition = disposition.toString();
                            if (disposition.indexOf('\r\n') == -1) {
                                content = content.toString();
                                let name = disposition.split(';')[1].split('=')[1];
                                name = name.substring(1, name.length - 1);
                                post[name] = content;
                            } else {
                                let [, name, filename] = disposition.split('\r\n')[0].split(';');
                                let filetype = disposition.split('\r\n')[1].split(':')[1];
                                name = name.split('=')[1];
                                name = name.substring(1, name.length - 1);
                                filename = filename.split('=')[1];
                                filename = filename.substring(1, filename.length - 1);
                                let path = (upload || '.') + '/' + uid + '-' + (++id);           
                                post[name] = {
                                    filename: filename,
                                    path: path,
                                    filetype: filetype,
                                    type: 0
                                };
                                if (upload && filename != '') {
                                    fs.writeFile(path, content, err => {
                                        if (err) {
                                            post[name].type = -1;
                                            post[name].error = err;
                                        } else {
                                            post[name].type = 1;
                                            //TODO: 考虑定时删除
                                        }
                                    });
                                } else {
                                    post[name].path = '';
                                    post[name].type = -1;
                                    post[name].content = content;
                                }
                            }
                        })
                    }
                } else if (req.headers['content-type'].indexOf('application/json') != -1) {
                    let json = data.toString();
                    try {
                        post = JSON.parse(json);
                    } catch (error) {
                        post = json;
                    }
                } else {
                    post = data.toString();
                }
            }
            req.POST = post;
            req.rawPOST = data;
            next();
        });
    }
}
```

感谢新创无际和新创乌鸡的多位开发者对SEP提供的建议与指导，更多的SEP开发中片段已与他们提前分享。
