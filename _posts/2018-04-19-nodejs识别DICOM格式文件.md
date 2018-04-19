---
layout:     post
title:      nodejs识别DICOM格式文件
subtitle:   nodejs文件流操作
date:       2018-04-19
author:     HYS
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - nodejs
    - fs
    - DICOM
---

>随便整理记录下项目中用到技巧点


# 前言

    项目中遇到上传附件功能，某些设备生成的DICOM文件本身是不带扩展名的，导致上传后程序无法使用文件名本身的后缀名截取方法来识别上传的附件的文件类型，此文旨在解决该问题

    DICOM文件特殊结构：文件开头会有128字节的导言，这部分数据没有内容。接着是4字节DICOM文件标识，存储着"DICM"，
    所以我们要做的就是读取上传的文件的内容，只读其中一部分,解析看这四个字节是不是'DICOM'即可;


# 使用nodejs同步方式读取文件内容

    //file是上传的文件对象
    export const fileExt = (name, dot = true, file) => {
        if (!name) {
            return '';
        }
        const idx = name.lastIndexOf('.');
        if (idx >= 0) {
            return (name.substr(dot ? idx : idx + 1) || '').toLowerCase();
        }

        //如果文件没有扩展名，尝试读取文件内容解析DICOM文件标识：
        const fd = fs.openSync(file.path, 'r');
        const buf = new Buffer(4);
        //DICOM文件特殊结构：文件开头会有128字节的导言，这部分数据没有内容。接着是4字节DICOM文件标识，存储着"DICM"
        fs.readSync(fd, buf, 0, 4, 128);
        let ext = '';
        const str = buf.toString();
        if (str === 'DICM') {
            ext = 'DICM';
            //console.log('upload file is a DICOM file,get the conent:', ext);
        }
        fs.closeSync(fd);
        return ext;    
    };


# 使用nodejs异步方式读取文件内容

    let ext = '';
    let str = '';
    //四个字节 file是上传的文件对象
    const readStream = fs.createReadStream(file.path, {start: 128, end: 131}); 

    const buffers = [];
    readStream.on('data', (buffer) => {        
        buffers.push(buffer);
    });
    readStream.on('end', () => {
        let ext = '';
        //直接字节数组转换成对应的字符串
        str = buffers.toString();
        //console.log('ext=========', str);
        if (str === 'DICM') {
            ext = 'DICM';
        }
        return ext;
    });
