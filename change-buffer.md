# Change Buffer 的目的？对 Insert Buffer升级，增加对 delete 的支持
Change Buffer 包含：
* Insert Buffer
* Delete Buffer
* Purge Buffer

1. Insert Buffer作用同之前
2. Delete Buffer 标记删除（非实际删除）
3. Purge Buffer 真正删除


