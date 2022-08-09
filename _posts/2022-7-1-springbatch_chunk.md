---
layout: post
title:  springbatch - chunk 단위 실행 
---
springbatch를 이용해서 데이터를 처리할때 데이터 사이즈가 큰경우 task 단위가 아닌 chunk를 이용해서 
데이터를 작은 사이즈로 read-process-write 를 반복적으로 처리에 견고하고 재처리 가능한 시스템으로 만들어 본다.


```java
@Bean(name = JOB_NAME + "_step")
@JobScope
public Step step() {
    log.info("step");
    return stepBuilderFactory.get(JOB_NAME + "Step")
        .<InputDto, OutputDto>chunk(CHUNK_SIZE)
        .reader(multiReader())
        .processor(processor())
        .writer(writer())
        .faultTolerant()
        .skip(IllegalStateException.class)
        .skipLimit(3)
        .listener(new NewsSkipListener())
        .listener(new NewsChunkListener())
        .exceptionHandler((context, throwable) -> {
            // send alarm or logging 
    })
    .build();

}
```

```java
    // 멀티파일리더 
    @Bean(name = JOB_NAME + "_multi_reader")
    @StepScope
    public MultiResourceItemReader<InputDto> multiReader() {

        MultiResourceItemReader<InputDto> multiReader = new MultiResourceItemReader<>();
        multiReader.setResources(inputResources);
        multiReader.setDelegate(reader());
        return multiReader;
    }
```

```java
    @Bean(name = JOB_NAME + "_reader")
    @StepScope
    public JsonItemReader<InputDto> reader() {
        return new JsonItemReaderBuilder<InputDto>()
            .jsonObjectReader(new JacksonJsonObjectReader<>(InputDto.class))
            .name("reader")
            .build();
    }
```

```java
    @Bean(name = JOB_NAME + "_processor")
    @StepScope
    public ItemProcessor<InputDto, OutputDto> processor(){
    return dto->{
        log.info("process item ");
        return OutputDto.builder().buil();
        };
    }
```

```java
    @Bean(name = JOB_NAME + "_writer")
    @StepScope
    public JpaItemWriter<OutputDto> writer() {
        return new JpaItemWriterBuilder<OutputDto>()
            .entityManagerFactory(entityManagerFactory)
            .build();
    }
```


chunk 단위 로깅 
```java
@Slf4j
public class NewsChunkListener implements org.springframework.batch.core.ChunkListener {
    @Override
    public void beforeChunk(ChunkContext context) {

    }

    @Override
    public void afterChunk(ChunkContext context) {
        StepContext stepContext = context.getStepContext();
        StepExecution stepExecution = stepContext.getStepExecution();
        log.info("##### after chunk read count:{}", stepExecution.getReadCount());
        log.info("##### after chunk skip count:{}", stepExecution.getProcessSkipCount());
        log.info("##### after chunk commit count:{}", stepExecution.getCommitCount());
    }

    @Override
    public void afterChunkError(ChunkContext context) {

    }
}
```

