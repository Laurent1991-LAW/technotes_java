# Stream API应用



stream以后，记得考虑是否需要：

- 去重：distinct() ;
- 判空并去除：filter() ;



```java
/* 以集合个体对象的某个属性为key，将集合分组分类，如以商品类型的父类id为key，对应的集合为value */
List<ItemCat> list = itemCatMapper.selectAllItemCat();
Map<Integer, List<ItemCat>> map = list.stream()
    								// 养成验证非空的习惯，避免后期空指针
                                      .filter(itemCat -> Objects.notNull(itemCat.getParentId()))
                                      .collect(Collectors.groupingBy(ItemCat::getParentId));
```



```java
/* 获取编码集合中后四位整数的最大值 */
// 获取当前前缀的附件编码集合，附件编码格式例如NEG202205150003
List<String> refs = fileMapper.findDocRefByPrefixe(prefixe+"%");
Integer max = refs.stream().map(ref -> {
                String suffix = ref.substring(ref.length() - 4);  // 获取编码后四位
                return Integer.parseInt(suffix);})
    				.collect(Collectors.maxBy(Integer::compare)).get();

```







