# 编译文件时使用 -s 选项，可以将目标可执行文件去掉调试信息
去掉 -g 选项

# c++ std::map定义为const类型时，不能使用[]。
因为，[]操作符默认如果查找不到就插入当前key。而此时类型是const，就无法完成。因此会编译不通过。

# iterator 使用错误导致coredump
```c
    if (_control.timingControl == CONTROL_TYPE_ON) {
        time_t now = REMOVE_DAY(time(NULL));
        control_item_t* target = NULL;
        for (auto it : _control.controls) {
            LOG_DEBUG("now {}, brightness {}", now, it.controlTime);
            if (now < it.controlTime)
                break;
            else
                target = &it;
        }
        if (target) {
            LOG_DEBUG("brightness {}", target->brightness);
            for (auto it : _data.current) {
            LOG_DEBUG("now brightness {} target {}", it.second->brightness, target->brightness);
                if (it.second->brightness != target->brightness) {
                    if (!set_brightness(0, it.second->eid, target->brightness)) {
                        LOG_DEBUG("set brightness {} failed", target->brightness);
                    } else {
                        LOG_DEBUG("set brightness {} success", target->brightness);
                    }
                }
            }
        }
    }
```
