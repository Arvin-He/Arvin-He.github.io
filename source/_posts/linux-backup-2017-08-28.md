---
title: Linux系统之镜像备份
date: 2017-08-28 13:51:53
tags: linux
categories: linux
---
### 计算文件拷贝的进度
```bash
# dd_process.sh
#! /bin/bash
####dd 命令反映进度####
if [ $# -ne 2 ];then
	echo "需要 盘符名 镜像名"
fi
	dupath=`du -m --total / --exclude=proc --exclude=media |grep 总用量 |cut -f 1`
	echo "根文件大小为$dupath M"
	let SIZE=$dupath+200
    while true
    do
    dusize=`du -hm /media/Lark/$1/$2 |cut -f 1`
    echo "生成文件大小为 $dusize M"
    if [ $dusize == $SIZE ]
        then
	echo "文件生成成功！"
        exit 0
    fi
    done
```

### 将根文件备份到u盘
```bash
#! /bin/bash
#################################
###  get img to u_disk                                 ### 
### 功能：将根文件备份到u盘                      ###  
#################################
#卸载备份区，保证/media/Lark下只挂载u盘
if [ -d /media/Lark/.linuxroot ];then
	/bin/umount /media/Lark/.linuxroot
fi

if [ $UID -ne 0 ];then
	echo "请切换root权限执行"
	exit -5
fi

if [ $# -ne 2 ]
	then
	echo "请输入 u盘名 镜像名称!"
	exit -7
fi

#检测出u盘个数及盘符名

LS_MEDIA_LARK=`ls /media/Lark`
SELECTED_DISK=
counter=0
for file in `ls /media/Lark`
do
counter=`expr $counter + 1`
done
#echo $counter

if [ $counter == 0 ]
	then
	#没有U盘插入，退出
	echo "没有插入u盘或者没有挂载盘符！！请检测u盘是否可正常识别!"
	exit -1
fi

#echo "LS_MEDIA_LARK is $LS_MEDIA_LARK "
DISK1=`echo $LS_MEDIA_LARK|awk -F ' ' '{print $1}'`
DISK2=`echo $LS_MEDIA_LARK|awk -F ' ' '{print $2}'`
#echo "DISK1 is $DISK1,DISK2 is $DISK2"


echo "检测目前根文件大小"
	dupath=`du -m --total / --exclude=proc --exclude=media |grep 总用量 |cut -f 1`
	echo "根文件大小为$dupath M"
	#size=2700
	#if [ $dupath -gt $size ];then
	#echo "注意：当前系统生成镜像大于3GB，生成镜像文件不可使用Lark升级工具烧写，但可使用原厂工具烧写"
	#	exit 0
	#fi
	#TODO :校验U盘可用空间，与根文件做对比
    echo "####生成根文件镜像####"
	let SIZE=$dupath+10
	echo "生成文件大小为$SIZE M"
	echo "步骤一:选择存放镜像的U盘"
	if false;then
	if [ -n "$DISK2" ] ;then
	echo "共检测出u盘 $counter个：请选择1.$DISK1 2.$DISK2"
	read CHOICE
	
	case $CHOICE in
		1) 
			echo "你选择的u盘为$DISK1"
			SELECTED_DISK=$DISK1
			;;
		2)
			echo "你选择的u盘为$DISK2"
			SELECTED_DISK=$DISK2
			;;
		*)
			echo "选择错误"
			exit -2
			;;
	esac
	else
			SELECTED_DISK=$DISK1
	fi
	fi
	SELECTED_DISK=$1
	#检测所选u盘是剩余空间
	
	DISK_SPACE=`df -hm /media/Lark/$SELECTED_DISK | sed -n "2p" | awk '{print $4}'`
	echo "剩余空间为 $DISK_SPACE M"
	
	if [ $SIZE -gt $DISK_SPACE ];then
	echo "根文件大小大于备份区最大空间，请删减可删减的文件进行备份"
	exit -3
	fi
	
	echo "步骤二:切换到U盘目录，且创建镜像文件，请耐心等待"
	cd  /media/Lark/$SELECTED_DISK

	if false;then
	#后台检测，一旦U盘断开，或者卸载，则退出此次操作
	{
		while true
		do
		CHECK_DISK=/media/Lark/$SELECTED_DISK
		if [ ! -d $CHECK_DISK ];then
			echo "u盘已断开，请检测其连接性并重新执行"
			echo
			exit -4
		fi
		if [ $FLAGS == 1 ];then
			exit 0
		fi
		done
	}&
	fi
	
	IMAGE_NAME=$2".tmp"
	
	echo  "步骤三:生成镜像名:$IMAGE_NAME"
		touch $IMAGE_NAME
	echo  "步骤四：生成指定大小空文件"
	dd if=/dev/zero of=$IMAGE_NAME bs=1M count=$SIZE
	
	if [ $? == 0 ]
		then
		echo  "步骤五：格式化镜像文件"
		mkfs.ext4 -F -L .linuxroot $IMAGE_NAME
	else
		echo "dd 操作失误"
		exit -1;
	fi
	
	if [ $? == 0 ]
		then
		echo "步骤六：挂载镜像文件到mnt目录"
	        mount -o loop $IMAGE_NAME /mnt
			
	    if [ $? == 0 ]
			then
			echo "步骤七：同步根分区到mnt"
			rsync -axv / /mnt
		fi 
	fi	
	
if [ $? == 0 ];then
	echo "同步完成"
	else
	echo "同步失败，请重新执行"
fi	

	if [ $? == 0 ]
		then
	#echo "请卸载U盘"
	cd /tmp
	umount /mnt
	if [ $? == 0 ]
		then
		 echo "挂载点已卸载"
	fi
	fi
	echo " 镜像已生成！"
	sleep 3
	#fuser -km /media/Lark/$SELECTED_DISK
	echo "镜像修正以便烧写"
	mv /media/Lark/$SELECTED_DISK/$IMAGE_NAME /media/Lark/$SELECTED_DISK/$2".img"
	umount /media/Lark/$SELECTED_DISK
	if [ $? == 0 ];then
		echo "$SELECTED_DISK 已卸载成功" 
		exit 0
		else
		echo "请卸载并拔除U盘"
	fi
```

### 系统备份
```bash
#! /bin/bash

if [ -d /media/Lark/.linuxroot ];then
	/bin/umount /media/Lark/.linuxroot
fi

echo "检测目前根文件大小"
	dupath=`du -m --total / --exclude=proc --exclude=media |grep 总用量 |cut -f 1`
    fssize=$dupath
	size=2500
	if [ $fssize -gt $size ];then
		echo "根文件大小大于备份区最大空间，请删减可删减的文件进行备份"
		exit 0
	fi

echo "检测系统一致性"
	/sbin/e2fsck -f -y  /dev/mmcblk0p6

if [ $? -ne 0 ];then
	echo "一致性检测失败，请检测重试"
	exit 0
fi

echo "扩展备份区到最大限额2.8G"
	/sbin/resize2fs /dev/mmcblk0p6 2600M
if [ $? -ne 0 ];then
	echo "扩大分区失败，请重试"
	exit 0
fi


echo "挂载系统备份区"

/bin/mount -o loop /dev/block/mtd/by-name/linuxfsbk /mnt

if [ $? -ne 0 ];then
	echo "挂载系统备份区失败，请重试"
	exit 0
fi

echo "同步当前系统"

/usr/bin/rsync -axv --delete / /mnt 

if [ $? -ne 0 ];then
	echo "同步当前系统失败，请重试"
	echo "卸载系统备份区"
	/bin/umount /mnt

	if [ $? -ne 0 ];then
		echo "卸载系统备份区失败，请重试"
		exit 0
	fi
	exit 0
fi
echo "同步完成"
echo "卸载系统备份区"
/bin/umount /mnt

if [ $? -ne 0 ];then
	echo "卸载系统备份区失败，请重试"
	exit 0
fi
echo "系统备份成功 "

sleep 2
```