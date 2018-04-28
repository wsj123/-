## mysql分表

关于大用户数据的分表的方法

### Markdown

根据php的crc32可以将字符串转化为数字并做一些其他处理，生成一个规则用来达到创建表，插入用户数据，根据用户的用户名称查询数据的规则,具体代码如下，用yii2实现

```markdown
// 生成表的规则
    private function rule($name) {
        $str = crc32($name);
        if($str < 0) {
            $hash = "0".substr(abs($str), 0, 1);
        }else{
            $hash = substr($str,0,2);
        }

        return 'user_'.$hash;
    }


    // 新增记录
    public function  actionAdd(){
        $name = Yii::$app->request->get('name');
        $tablename = $this->rule($name);
        // 创建表
        $res = $this->ct($tablename);
        $password = md5('123');
        $this->inn($name,$password,$tablename);

    }


    // 创建表
    private function ct($tablename) {
        $sql = "CREATE TABLE `{$tablename}` (
              `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
              `name` varchar(64) NOT NULL DEFAULT '' COMMENT '用户名',
              `password` varchar(128) NOT NULL DEFAULT '' COMMENT '密码',
              PRIMARY KEY (`id`)
              ) ENGINE=InnoDB DEFAULT CHARSET=utf8;";


        $db = Yii::$app->db;
        // 查看表是否存在，不存在则创建
        $ta=$db->createCommand("SHOW TABLES LIKE '".$tablename."'")->queryAll();
        if($ta==null)
        {
            $res = $db->createCommand($sql)
                ->execute();
        }

        return true;
    }

    // 插入用户记录
    private function inn($name,$password,$tablename) {
        $sql = "insert into $tablename(name,password) values ('$name','$password');";
        $db = Yii::$app->db;
        $res = $db->createCommand($sql)
            ->execute();
        return $res;
    }

    // 根据用户名查询记录
    public function  actionLook(){
        header('content-type:text/html;charset=utf-8');
        $name = Yii::$app->request->get('name');
        $tablename = $this->rule($name);
        $sql = "select *from {$tablename} where name='$name'";
        echo $sql;
        echo '<hr>';
        $res = Yii::$app->db->createCommand($sql)
            ->queryOne();
        var_dump($res);exit;
    }
    运行效果如下:
    select *from user_54 where name='二狗子'
array(3) { ["id"]=> string(1) "1" ["name"]=> string(9) "二狗子" ["password"]=> string(32) "202cb962ac59075b964b07152d234b70" }
```
