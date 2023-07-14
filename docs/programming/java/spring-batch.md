---
title: Spring Batch chunk 반복 실행
layout: default
parent: Java
grand_parent: Programming
---


## chunk 반복실행 되지 않음    
   
처음엔 tasklet으로 작성하였지만 대량처리를 할 경우 tasklet보다는 chunk를 이용하여 chuck_size에 맞게 나눠처리하도록 하는 것이 자원에 과부화가 없고 좋다고 하여 chunk로 변경했다.

```
//BatchConfiguration.java

import com.education.education2.entity.user.UserEntity;
import com.education.education2.mapper.UserMapper;
import org.springframework.batch.core.Job;
import org.springframework.batch.core.Step;
import org.springframework.batch.core.StepContribution;
import org.springframework.batch.core.job.builder.JobBuilder;
import org.springframework.batch.core.repository.JobRepository;
import org.springframework.batch.core.scope.context.ChunkContext;
import org.springframework.batch.core.step.builder.StepBuilder;
import org.springframework.batch.item.Chunk;
import org.springframework.batch.item.ItemProcessor;
import org.springframework.batch.item.ItemWriter;
import org.springframework.batch.item.support.ListItemReader;
import org.springframework.batch.repeat.RepeatStatus;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.transaction.PlatformTransactionManager;

import java.time.LocalDateTime;

@Configuration
public class BatchConfiguration {

    @Autowired
    private UserMapper userMapper;

    @Bean
    public Job job(JobRepository jobRepository, Step step, Step step2){
        return new JobBuilder("job", jobRepository)
                .start(step2)
                .next(step)
                .build();
    }

    @Bean
    public Step step(JobRepository jobRepository, PlatformTransactionManager transactionManager) {
        return new StepBuilder("step", jobRepository)
                .<UserEntity, UserEntity>chunk(2)
                .transactionManager(transactionManager)
                .reader(new ListItemReader<UserEntity>(userMapper.findUserAll()))
                .processor(new ItemProcessor<UserEntity, UserEntity>() {
                    @Override
                    public UserEntity process(UserEntity user) throws Exception {
                        if (user.getIsDlt().equals("Y") && user.getUdtDtm().plusYears(1).compareTo(LocalDateTime.now()) < 0) {
                            System.out.printf("Delete User: " + user.getUsrId() + '\n');
                            return user;
                        }
                        return null;
                    }
                })
                .writer(new ItemWriter<UserEntity>() {
                    @Override
                    public void write(Chunk<? extends UserEntity> user) throws Exception {
                        System.out.printf("Successfully Delete Users: " + user + "\n");
                    }
                })
                .build();
    }

    @Bean
    public Step step2(JobRepository jobRepository, PlatformTransactionManager transactionManager) {
        return new StepBuilder("step", jobRepository)
                .tasklet((StepContribution contribution, ChunkContext chunkContext) -> {
                    System.out.printf("hi hello" + '\n');
                    Thread.sleep(10);
                    System.out.printf("=======doooneeee======" + "\n");
                    return RepeatStatus.FINISHED;
                }, transactionManager)
                .build();
    }
}
```
   
batch는 하나의 작업을 일괄처리되도록 작성한 코드일 뿐, 특정시간에 동작하도록 하기 위해서는 스케줄러로 시간을 지정해줘야 하는 것 같다. 스케줄러로 사용되는 건 Scheduler와 Quarts 두개가 있는데 퀄츠는 어려워 보여 스케줄러로 일단 작업했다.

```
//BatchScheduler.java

import lombok.extern.slf4j.Slf4j;
import org.springframework.batch.core.JobParameters;
import org.springframework.batch.core.JobParametersBuilder;
import org.springframework.batch.core.JobParametersInvalidException;
import org.springframework.batch.core.launch.JobLauncher;
import org.springframework.batch.core.repository.JobExecutionAlreadyRunningException;
import org.springframework.batch.core.repository.JobInstanceAlreadyCompleteException;
import org.springframework.batch.core.repository.JobRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;
import org.springframework.transaction.PlatformTransactionManager;

import java.util.Calendar;

@Slf4j
@Component
public class BatchScheduler {

    @Autowired
    private JobLauncher jobLauncher;

    @Autowired
    private BatchConfiguration batchConfiguration;
    private JobRepository jobRepository;

    private PlatformTransactionManager transactionManager;

//    @Scheduled(cron = "0/1 * * * * ?") //1초마다 실행
    @Scheduled(cron = "0/10 * * * * ?")
    public void runJob() {
        try{
            JobParameters jobParameters = new JobParametersBuilder()
                .addDate("timestamp", Calendar.getInstance().getTime())
                .toJobParameters();
            jobLauncher.run(batchConfiguration.job(jobRepository, batchConfiguration.step(jobRepository, transactionManager), batchConfiguration.step2(jobRepository, transactionManager)), jobParameters);
        } catch(JobExecutionAlreadyRunningException | JobInstanceAlreadyCompleteException
                | JobParametersInvalidException | org.springframework.batch.core.repository.JobRestartException e) {
            log.error(e.getMessage());
        }
    }
}
```
   
하지만 chunk로 작업을 하니 스케줄러가 작동을 해도 chunk인 step은 프로젝트 실행할 때 한 번 작동 후 그 뒤로는 tasklet만 작동한다..   

![spring-batch1](/assets/images/spring-batch1.png)
   
[해결]   

chunk가 한번만 돌고 그 뒤로 실행이 안되서 진심 batch에 대한 정보란 정보는 다 본 거 같은데 해결을 못해서 [StackOverFlow]에 글을 올렸는데 ListItemReader은 @Bean과 함께 @JobScope이나 @StepScope을 사용해야 한다는 답변이 달렸다.   

![spring-batch2](/assets/images/spring-batch2.png)

   
그래서 ListItemReader를 사용하는 Step에 @JobScope 어노테이션을 추가 했더니 스케줄러에 설정한 대로 10마다 해당 batch가 실행된다.   

```
@Bean
@StepScope
public Step step(JobRepository jobRepository, PlatformTransactionManager transactionManager) {
    return new StepBuilder("step", jobRepository)
            .<String, String>chunk(5)
            .reader(new ListItemReader<>(Arrays.asList("item1", "item2", "item3", "item4", "item5")))
            .processor((ItemProcessor<String, String>) item -> {
                Thread.sleep(300); // 0.3초 delay
                System.out.println("item = " + item);
                return "seohae : " + item;
            })
            .writer(new ItemWriter<String>() {
                @Override
                public void write(Chunk<? extends String> chunk) throws Exception {
                    System.out.println("items = " + chunk);
                }
            })
            .transactionManager(transactionManager)
            .build();
}
```
   
[결과]  

![spring-batch3](/assets/images/spring-batch3.png)

[StackOverFlow]: https://stackoverflow.com/questions/76677030/when-i-used-scheduled-the-chunk-in-spring-batch-is-not-working "stackoverflow 문의 답변"