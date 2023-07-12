---
title: Spring batch error
layout: default
parent: Spring Boot
grand_parent: Error
nav_order: 6
---

{: .warning }
 Table "BATCH_JOB_INSTANCE" not found


Spring batch 코드를 작성한 후 실행을 했지만 위와 같은 오류가 발생함.

```
//** BatchConfiguration.java */

import org.springframework.batch.core.Job;
import org.springframework.batch.core.Step;
import org.springframework.batch.core.StepContribution;
import org.springframework.batch.core.job.builder.JobBuilder;
import org.springframework.batch.core.repository.JobRepository;
import org.springframework.batch.core.scope.context.ChunkContext;
import org.springframework.batch.core.step.builder.StepBuilder;
import org.springframework.batch.repeat.RepeatStatus;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.transaction.PlatformTransactionManager;

@Configuration
public class BatchConfiguration {

    @Bean
    public Job job(JobRepository jobRepository, Step step){
        return new JobBuilder("job", jobRepository)
                .start(step)
                .build();
    }

    @Bean
    public Step step(JobRepository jobRepository, PlatformTransactionManager transactionManager) {
        return new StepBuilder("step", jobRepository)
                .tasklet((StepContribution contribution, ChunkContext chunkContext) -> {
                    System.out.printf("hi hello");
                    return RepeatStatus.FINISHED;
                }, transactionManager)
                .build();
    }
}
```

application.yml 파일에 [아래 코드]를 입력하면 해당 오류 없이 정상적으로 실행된다.


```
//** application.yml */
//properties의 경우
spring.batch.initialize-schema=always

// yml의 경우
spring:
  batch:
    jdbc:
      initialize-schema: always
```

[아래 코드]: https://hangjastar.tistory.com/231 "Batch 테이블 not found오류 해결"
