#!/bin/bash
LOG_FILE="/var/log/simplified_system_inspection.log"  #定义巡检日志文件目录位置
# 检查日志文件所在目录是否存在，若不存在则尝试创建
if [! -d "$(dirname "$LOG_FILE")" ]; then
    mkdir -p "$(dirname "$LOG_FILE")"
    if [ $? -ne 0 ]; then
        echo "无法创建日志目录，脚本终止。"
        exit 1
    fi
fi

# 获取当前时间
CURRENT_TIME=$(date +"%Y-%m-%d %H:%M:%S")

# 记录脚本开始执行的时间
echo "[$CURRENT_TIME] System Inspection Script Started" >> $LOG_FILE

# 系统基本信息
echo "=== 系统基本信息 ===" >> $LOG_FILE
echo "系统类型: $(uname -s)" >> $LOG_FILE
echo "系统版本: $(cat /etc/redhat-release)" >> $LOG_FILE
echo "内核版本: $(uname -r)" >> $LOG_FILE
echo "主机名: $(hostname)" >> $LOG_FILE
echo "当前时间: $(date)" >> $LOG_FILE

# CPU 信息
echo "=== CPU 信息 ===" >> $LOG_FILE
echo "逻辑 CPU 核数: $(grep -c processor /proc/cpuinfo)" >> $LOG_FILE
echo "CPU 型号: $(grep'model name' /proc/cpuinfo | head -1 | cut -d ':' -f 2 | sed's/^[ \t]*//')" >> $LOG_FILE
echo "CPU 负载 (1 分钟): $(uptime | awk -F 'load average:' '{print $2}' | cut -d ',' -f 1 | sed's/^[ \t]*//')" >> $LOG_FILE# 内存信息

echo "=== 内存信息 ===" >> $LOG_FILE
echo "总内存: $(free -h | grep Mem | awk '{print $2}')" >> $LOG_FILE
echo "已使用内存: $(free -h | grep Mem | awk '{print $3}')" >> $LOG_FILE
echo "空闲内存: $(free -h | grep Mem | awk '{print $4}')" >> $LOG_FILE

# 磁盘信息
echo "=== 磁盘信息 ===" >> $LOG_FILE
df -h >> $LOG_FILE
# 网络信息
echo "=== 网络信息 ===" >> $LOG_FILE
echo "IP 地址: $(hostname -I | awk '{print $1}')" >> $LOG_FILE
echo "网关: $(ip route | grep default | awk '{print $3}')" >> $LOG_FILE
echo "DNS: $(grep 'nameserver' /etc/resolv.conf | awk '{print $2}')" >> $LOG_FILE
if ping -c 2 -w 2 www.baidu.com &> /dev/null; then
    echo "网络连接: 正常" >> $LOG_FILE
else
    echo "网络连接: 异常" >> $LOG_FILE
fi

# 记录脚本结束执行的时间
CURRENT_TIME=$(date +"%Y-%m-%d %H:%M:%S")
echo "[$CURRENT_TIME] System Inspection Script Ended" >> $LOG_FILE
