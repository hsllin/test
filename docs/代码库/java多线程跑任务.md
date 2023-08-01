
```java
ExecutorService executor = Executors.newFixedThreadPool(10);
 List<AttendanceClockGroupSummaryVo> groupSummaryVoList = new ArrayList<>();  
  
        List<CompletableFuture<AttendanceClockGroupSummaryVo>> futures = new ArrayList<>();  
        for (String queryDate : queryDateList) {  
            futures.add(CompletableFuture.supplyAsync(() -> {  
                AttendanceClockGroupSummaryVo summaryVo = new AttendanceClockGroupSummaryVo();  
                //搜索条件  
                summaryVo = dealGroupDayCountData(queryDate, depId, userId);  
                return summaryVo;  
            }, executor));  
        }  
        //等待全部完成  
        CompletableFuture.allOf(futures.toArray(new CompletableFuture[0])).join();  
  
//获取内容  
        for (CompletableFuture<AttendanceClockGroupSummaryVo> future : futures) {  
            try {  
                groupSummaryVoList.add(future.get());  
            } catch (InterruptedException | ExecutionException e) {  
                e.printStackTrace();  
            }  
        }  
        return groupSummaryVoList;
```
