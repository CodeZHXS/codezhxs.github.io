[参考文档](https://hydro.js.org/docs/)

## 网页注册

```bash
# 关闭
hydrooj cli user setPriv 0 0
pm2 restart hydrooj
```

```bash
# 启动
hydrooj cli user setPriv 0 8
pm2 restart hydrooj
```

## 修改页面

在URL前加入 `view-source:` 查看 `data-page` 的值，在[这里](https://github.com/hydro-dev/Hydro/blob/master/packages/ui-default/templates/manage_setting.html)找到对应的html文件

cd到 `~/.hydro/addons/addon/templates` 进行修改。