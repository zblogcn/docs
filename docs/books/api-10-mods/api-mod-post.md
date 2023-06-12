## post 文章管理 API

| act 方法 | 请求方式   | 参数                                                     | 鉴权               |
| -------- | ---------- | ------------------------------------------------------- | ----------------- |
| `get`    | GET / POST | `获取文章`                                                | 非公开文章需鉴权   |
|          |            |参数 id:文章 ID
|          |            |参数 with_relations:追加的关联对象  | 例：mod=post&act=get&id=2&with_relations=Author 同时输出 Author 对象
|          |            |1.7.2 新增参数 viewnums:同时刷新浏览计数 | 例：mod=post&act=get&id=2&viewnums=1
|
| `post`   | POST       | `新建或编辑文章`                                            |必须               |
|          |            |`Post`表单字段定义 附：**「示例 1」**
|
| `delete` | GET / POST | `删除文章`                                                  | 必须               |
|          |            |$_REQUEST['id'] 文章 `id`
|
| `list`   | GET / POST | `获取文章列表` | 未鉴权请求数量受限为<br/>每页面展示数量 |
|          |            |$_REQUEST 参数定义如下
|          |            | `cate_id`, `tag_id`, `auth_id`, `type`, `date`, `manage`, `search` |已鉴权有后台管理权限为<br/>后台每页面展示数量 |
|          |            |                                                          
|          |            |`act=list`方法共通参数见：[约束与过滤](books/api-05-design?id=约束与过滤 "约束与过滤")
|          |            |参数 with_relations:追加的关联对象  | 例：mod=post&act=get&id=2&with_relations=Author 同时输出 Author 对象
|          |            |1.7.2.3045 增加 参数 with_subcate:可以在分类列表输出子分类的文章 | 例：mod=post&act=list&cate_id=2&with_subcate=1
|

**示例 1**：

<details>
<summary>post 新建或发布文章的$_POST参数示范：（点击展开）</summary>

```php
$_POST['ID'] 为 0 是新建
$_POST['Title']
$_POST['Alias']
$_POST['Type'] 为 0 是文章，1 是 page 页面
$_POST['AuthorID']
$_POST['CateID'] 如果没有提供 CateID,可提供 CateName
$_POST['Intro']
$_POST['Content'] 
$_POST['Tag']
$_POST['PostTime']
$_POST['Status'] 状态
```
注：对于发布文章，额外提供一个`CateName`字段可用来代替`CateID`指定分类，前提是存在以该字段值命名的分类；也可以使用`category`模块内的接口实现自动创建分类等操作；
</details>
