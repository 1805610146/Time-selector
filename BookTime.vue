<template>
  <div>
    <div class="tip">
      <span>质量创新中心</span>
      <span>建议时长：60min</span>
      <span>参观上限：60min</span>
    </div>
    <div class="picker" @mousemove="onMouseMove" @mouseup="onMouseUp">
      <el-tooltip placement="top" effect="light" v-for="(item, index) in timePicker" :key="index"
        :show-after="showAfter()" :content="getTimeContent(index)">
        <div class="picker-item" :class="getitemClass(item)" @mousedown="onDrag(index)">
          <div class="time-item" v-if="index % 4 === 0">{{ timeList[index] }}</div>
          <div class="time-item-last" v-if="index === 35">{{ timeList[index + 1] }}</div>
        </div>
      </el-tooltip>
    </div>
  </div>
</template>

<script setup lang="ts">
import { reactive, ref } from 'vue';
import { ElMessage } from 'element-plus';

// 0代表空闲，1代表不可预定，2代表正在关注
const timePicker = reactive<number[]>([0, 0, 0, 0, 1, 1, 1, 1, 0, 2, 2, 2, 2, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0]);
// 时间表数组
const timeList = reactive<string[]>([]);
// 拖动状态
const isDragging = ref(false);
// 存储拖动起始索引
const dragStartIndex = ref<number | null>(null);
const dragEndIndex = ref<number | null>(null);


// 计算时间槽
const startHour = 9;
const endHour = 18;
const interval = 15;
for (let hour = startHour; hour <= endHour; hour++) {
  for (let minute = 0; minute < 60; minute += interval) {
    // 单独添加最后一个时间槽
    if (hour === endHour && minute === 0) {
      const time = `${hour}:${minute < 10 ? '0' : ''}${minute}`;
      timeList.push(time);
      break;
    } else if (hour < endHour) {
      const time = `${hour}:${minute < 10 ? '0' : ''}${minute}`;
      timeList.push(time);
    }
  }
}
// 时间槽状态选择不同的样式
const getitemClass = (item: number) => {
  if (item === 0) {
    return 'timeitem-free';
  } else if (item === 1) {
    return 'timeitem-disabled';
  } else {
    return 'timeitem-active';
  }
}

// 延迟显示时间提示
const showAfter = () => {
  return 150;
}

// 获取悬浮文本
const getTimeContent = (index: number) => {
  if (timePicker[index] === 0) {
    return timeList[index] + ' - ' + timeList[index + 1] + ' 可预约';
  } else if (timePicker[index] === 1) {
    return timeList[index] + ' - ' + timeList[index + 1] + ' 不可预定';
  } else {
    const minIndex = timePicker.indexOf(2);
    const maxIndex = timePicker.lastIndexOf(2);
    return timeList[minIndex] + ' - ' + timeList[maxIndex + 1] + ' 正在关注';
  }
}

// 鼠标下落事件
const onDrag = (e: number) => {
  if (timePicker[e] === 0) {
    timePicker.forEach((item, index) => {
      if (item == 2) {
        timePicker[index] = 0;
      }
    });
    timePicker[e] = 2;
  } else if (timePicker[e] === 2) {
    timePicker[e] = 0;
    let flag = false;//判断是否出现正在关注的
    for (let i = 0; i < timePicker.length; i++) {
      if (timePicker[i] == 2) {
        flag = true;
      }
      if (timePicker[i] != 2 && flag) {
        for (let j = i + 1; j < timePicker.length; j++) {
          if (timePicker[j] == 2) {
            timePicker[j] = 0;
          }
        }
        break;
      }
    }
  } else {
    ElMessage({
      message: '不可预约',
      type: 'error',
      plain: true,
    });
    return;
  }
  dragStartIndex.value = e;
  isDragging.value = true;
};
// 鼠标移动事件
const onMouseMove = (e: MouseEvent) => {
  if (isDragging.value && dragStartIndex.value !== null) {
    //判断鼠标是否拖到了另一格子
    if (dragEndIndex.value != getIndexFromEvent(e)) {
      dragEndIndex.value = getIndexFromEvent(e);
      if (dragEndIndex.value !== null) {
        // 这里可以添加更多逻辑，例如根据拖动方向更新状态
        if (timePicker[dragStartIndex.value] === 2) {
          setTimePicker(true, dragStartIndex.value, dragEndIndex.value);
        } else if (timePicker[dragStartIndex.value] === 0) {
          setTimePicker(false, dragStartIndex.value, dragEndIndex.value);
        }
      }
    }
  }
};

const errorMessage = (message: string) => {
  ElMessage({
    message,
    type: 'error',
    plain: true,
    offset: 85,
  });
}
//事件槽的设置
function setTimePicker(select: boolean, startIndex: number, endIndex: number) {
  let minIndex = Math.min(startIndex, endIndex);
  let maxIndex = Math.max(startIndex, endIndex);
  if (select) {
    if (maxIndex - minIndex > 3) {
      errorMessage('预约时间不能超过60min');
      if (startIndex > endIndex) {
        minIndex = maxIndex - 3;
      } else {
        maxIndex = minIndex + 3;
      }
      isDragging.value = false;
      dragStartIndex.value = null;
      dragEndIndex.value = null;
    }

    for (let i = minIndex; i <= maxIndex; i++) {
      if (timePicker[i] != 1 && timePicker[i] != 3) {
        timePicker[i] = 2;
      } else {
        errorMessage('不可预约');
        isDragging.value = false;
        dragStartIndex.value = null;
        dragEndIndex.value = null;
      }
    }
  } else {
    for (let i = minIndex; i <= maxIndex; i++) {
      if (timePicker[i] != 1 && timePicker[i] != 3) {
        timePicker[i] = 0;
      } else {
        errorMessage('不可预约');
        break;
      }
    }
  }
}

// 鼠标释放事件
const onMouseUp = () => {
  isDragging.value = false;
  dragStartIndex.value = null;
  dragEndIndex.value = null;
};
// 根据鼠标事件计算索引
const getIndexFromEvent = (e: MouseEvent): number | null => {
  const target = e.target as HTMLElement;
  const pickerItems = document.querySelectorAll('.picker-item');
  for (let i = 0; i < pickerItems.length; i++) {
    if (pickerItems[i].contains(target)) {
      return i;
    }
  }
  return null;
};
</script>

<style lang="scss" scoped>
.tip {
  display: flex;
  justify-content: space-between;
  align-items: center;
  width: 100%;
  height: 30px;
  color: #999;
  font-size: 14px;
  margin-top: 50px;
}

.picker {
  display: flex;
  width: 100%;
  max-width: 900px;
  min-width: 600px;
  height: 25px;
  background-color: #E4F8FF;
  border: 1px solid #0066C4;
  margin: 20px 0 50px 0;

  .picker-item {
    position: relative;
    width: 25px;
    flex: 1;
    height: 100%;
    border-right: 1px solid #0066C4;
    user-select: none;

    &:last-child {
      border-right: none;
    }
  }

  .time-item {
    position: absolute;
    bottom: -100%;
    left: -70%;

  }

  .time-item-last {
    position: absolute;
    bottom: -100%;
    right: -70%;
  }
}

.timeitem-free {
  background-color: #E4F8FF;
}

.timeitem-disabled {
  background-color: #C1C1C1;
}

.timeitem-active {
  background-color: #FFA770;
}
</style>
