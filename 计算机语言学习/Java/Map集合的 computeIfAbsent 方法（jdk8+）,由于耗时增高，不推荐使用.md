## Map集合的 computeIfAbsent 方法（jdk8+）,由于耗时增高，不推荐使用

```java
    @Test
    public void test2(){
        long start = System.nanoTime();
        Map<Integer,List<String>> map = new HashMap<>();
        for (int i = 0; i < 10; i++) {
            if (map.get(i) == null){
                map.put(i,new ArrayList<>());
            }
            map.get(i).add("test" + i);
        }

        long end = System.nanoTime();
        System.out.println(end-start);
// normal： 95625 ns 
        long start1 = System.nanoTime();
        Map<Integer,List<String>> map1 = new HashMap<>();
        for (int i = 0; i < 10; i++) {
            map1.computeIfAbsent(i, k -> new ArrayList<>());
            map1.get(i).add("test" + i);
        }
        long end1 = System.nanoTime();
        System.out.println(end1-start1);
      // computeIfAbsent : 88291834 ns
    }
// 由上实验可知 computeIfAbsent 性能效果并不好，只是在代码整洁方面比较好。
```