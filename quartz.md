

- job相关

  - qrtz_job_details 存储配置的job详细信息

- trigger相关

  - qrtz_triggers 触发器基本信息
    - qrtz_simple_triggers 相当于定时触发(fixDelay)
    - qrtz_cron_triggers cron方式触发（cron）
    - qrtz_blob_triggers （少用）
    - qrtz_simprop_triggers （少用）

- 运行时相关

  - qrtz_fired_triggers 已触发的Trigger相关的状态信息
  - qrtz_scheduler_state 集群中每个实例的当前状态
  - qrtz_locks 集群锁



一个trigger对应一个job

一般cronTrigger就足够用

misfire policy

- MISFIRE_INSTRUCTION_SMART_POLICY = 0
  - 对应CronTrigger MISFIRE_INSTRUCTION_FIRE_ONCE_NOW = 1，更新下次执行时间，之前未执行的当前立刻执行一次
  - 对应SimpleTrigger 可看以下SimpleTrigger updateAfterMisfire的代码



- MISFIRE_INSTRUCTION_IGNORE_MISFIRE_POLICY = -1，截至目前为止如果未执行成功8次，则启动后会立刻执行8次
- CronTrigger MISFIRE_INSTRUCTION_DO_NOTHING = 2，只更新下次执行时间，之前未执行的忽略



CronTrigger updateAfterMisfire

```java
@Override
    public void updateAfterMisfire(org.quartz.Calendar cal) {
        int instr = getMisfireInstruction();

        if(instr == Trigger.MISFIRE_INSTRUCTION_IGNORE_MISFIRE_POLICY)
            return;

        if (instr == MISFIRE_INSTRUCTION_SMART_POLICY) {
            instr = MISFIRE_INSTRUCTION_FIRE_ONCE_NOW;
        }

        if (instr == MISFIRE_INSTRUCTION_DO_NOTHING) {
            Date newFireTime = getFireTimeAfter(new Date());
            while (newFireTime != null && cal != null
                    && !cal.isTimeIncluded(newFireTime.getTime())) {
                newFireTime = getFireTimeAfter(newFireTime);
            }
            setNextFireTime(newFireTime);
        } else if (instr == MISFIRE_INSTRUCTION_FIRE_ONCE_NOW) {
            setNextFireTime(new Date());
        }
    }
```



SimpleTrigger updateAfterMisfire

```java
@Override
    public void updateAfterMisfire(Calendar cal) {
        int instr = getMisfireInstruction();
        
        if(instr == Trigger.MISFIRE_INSTRUCTION_IGNORE_MISFIRE_POLICY)
            return;
        
        if (instr == Trigger.MISFIRE_INSTRUCTION_SMART_POLICY) {
            if (getRepeatCount() == 0) {
                instr = MISFIRE_INSTRUCTION_FIRE_NOW;
            } else if (getRepeatCount() == REPEAT_INDEFINITELY) {
                instr = MISFIRE_INSTRUCTION_RESCHEDULE_NEXT_WITH_REMAINING_COUNT;
            } else {
                // if (getRepeatCount() > 0)
                instr = MISFIRE_INSTRUCTION_RESCHEDULE_NOW_WITH_EXISTING_REPEAT_COUNT;
            }
        } else if (instr == MISFIRE_INSTRUCTION_FIRE_NOW && getRepeatCount() != 0) {
            instr = MISFIRE_INSTRUCTION_RESCHEDULE_NOW_WITH_REMAINING_REPEAT_COUNT;
        }

        if (instr == MISFIRE_INSTRUCTION_FIRE_NOW) {
            setNextFireTime(new Date());
        } else if (instr == MISFIRE_INSTRUCTION_RESCHEDULE_NEXT_WITH_EXISTING_COUNT) {
            Date newFireTime = getFireTimeAfter(new Date());
            while (newFireTime != null && cal != null
                    && !cal.isTimeIncluded(newFireTime.getTime())) {
                newFireTime = getFireTimeAfter(newFireTime);

                if(newFireTime == null)
                    break;
                
                //avoid infinite loop
                java.util.Calendar c = java.util.Calendar.getInstance();
                c.setTime(newFireTime);
                if (c.get(java.util.Calendar.YEAR) > YEAR_TO_GIVEUP_SCHEDULING_AT) {
                    newFireTime = null;
                }
            }
            setNextFireTime(newFireTime);
        } else if (instr == MISFIRE_INSTRUCTION_RESCHEDULE_NEXT_WITH_REMAINING_COUNT) {
            Date newFireTime = getFireTimeAfter(new Date());
            while (newFireTime != null && cal != null
                    && !cal.isTimeIncluded(newFireTime.getTime())) {
                newFireTime = getFireTimeAfter(newFireTime);

                if(newFireTime == null)
                    break;
                
                //avoid infinite loop
                java.util.Calendar c = java.util.Calendar.getInstance();
                c.setTime(newFireTime);
                if (c.get(java.util.Calendar.YEAR) > YEAR_TO_GIVEUP_SCHEDULING_AT) {
                    newFireTime = null;
                }
            }
            if (newFireTime != null) {
                int timesMissed = computeNumTimesFiredBetween(nextFireTime,
                        newFireTime);
                setTimesTriggered(getTimesTriggered() + timesMissed);
            }

            setNextFireTime(newFireTime);
        } else if (instr == MISFIRE_INSTRUCTION_RESCHEDULE_NOW_WITH_EXISTING_REPEAT_COUNT) {
            Date newFireTime = new Date();
            if (repeatCount != 0 && repeatCount != REPEAT_INDEFINITELY) {
                setRepeatCount(getRepeatCount() - getTimesTriggered());
                setTimesTriggered(0);
            }
            
            if (getEndTime() != null && getEndTime().before(newFireTime)) {
                setNextFireTime(null); // We are past the end time
            } else {
                setStartTime(newFireTime);
                setNextFireTime(newFireTime);
            } 
        } else if (instr == MISFIRE_INSTRUCTION_RESCHEDULE_NOW_WITH_REMAINING_REPEAT_COUNT) {
            Date newFireTime = new Date();

            int timesMissed = computeNumTimesFiredBetween(nextFireTime,
                    newFireTime);

            if (repeatCount != 0 && repeatCount != REPEAT_INDEFINITELY) {
                int remainingCount = getRepeatCount()
                        - (getTimesTriggered() + timesMissed);
                if (remainingCount <= 0) { 
                    remainingCount = 0;
                }
                setRepeatCount(remainingCount);
                setTimesTriggered(0);
            }

            if (getEndTime() != null && getEndTime().before(newFireTime)) {
                setNextFireTime(null); // We are past the end time
            } else {
                setStartTime(newFireTime);
                setNextFireTime(newFireTime);
            } 
        }

    }
```

