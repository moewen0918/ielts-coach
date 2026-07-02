# 🎯 雅思精读 AI 双引擎复盘工作台 Gemini and DeepSeek

这是一个专门为了雅思阅读复盘设计的 AI 工具。配合油猴脚本，可以在任何做题网站上实现“鼠标划词 -> 一键打包投递解析”。

## 🚀 任何人如何开始使用？

### 第一步：安装浏览器快递员（油猴插件）
要实现一键划词打包，你的浏览器需要安装 [Tampermonkey（油猴插件）](https://www.tampermonkey.net/)。

### 第二步：安装我们的专属一键打包脚本
1. 安装好油猴插件后，点击创建一个新脚本。
2. 将以下代码复制并保存到你的油猴脚本中即可：
 // ==UserScript==
// @name         雅思精读 批量语料打包器(双引擎Pro Max版)
// @namespace    http://tampermonkey.net/
// @version      13.0
// @description  精准抓取阅读Part、文章标题、高亮混合语料，并百分百安全投递至双引擎 Pro Max 全能工作台
// @author       Gemini
// @match        *://*/*
// @match        file:///*
// @include      file://*
// @include      *
// @run-at       document-end
// @grant        none
// ==/UserScript==

(function() {
    'use strict';

    window.addEventListener('load', function() {
        setTimeout(createBatchButton, 2000);
    });

    function createBatchButton() {
        if (document.getElementById('tamper-batch-btn')) return;

        let btn = document.createElement('button');
        btn.id = 'tamper-batch-btn';
        btn.innerHTML = '📦 <br>打包解析';
        btn.style.position = 'fixed';
        btn.style.top = '60%';
        btn.style.right = '20px';
        btn.style.transform = 'translateY(-50%)';
        btn.style.zIndex = '2147483647';
        btn.style.padding = '12px 8px';
        btn.style.background = 'rgba(52, 199, 89, 0.95)';
        btn.style.backdropFilter = 'blur(10px)';
        btn.style.color = '#fff';
        btn.style.border = '1px solid rgba(255,255,255,0.3)';
        btn.style.borderRadius = '14px';
        btn.style.boxShadow = '0 8px 24px rgba(52, 199, 89, 0.4)';
        btn.style.cursor = 'pointer';
        btn.style.fontWeight = 'bold';
        btn.style.fontSize = '13px';
        btn.style.transition = 'all 0.2s ease';

        document.body.appendChild(btn);

        btn.onclick = function() {
            // 1. 利用浏览器网址栏的参数进行高精度章节匹配
            let partText = "Part 1";
            let currentUrl = window.location.href;

            if (currentUrl.includes('examId=p2') || currentUrl.includes('dataKey=p2')) {
                partText = 'Part 2';
            } else if (currentUrl.includes('examId=p3') || currentUrl.includes('dataKey=p3')) {
                partText = 'Part 3';
            } else {
                let pageText = document.body.innerText;
                if(pageText.includes('READING PASSAGE 2')) partText = 'Part 2';
                else if(pageText.includes('READING PASSAGE 3')) partText = 'Part 3';
            }

            // 2. 精准抓取文章标题
            let titleText = "未知文章";
            let titleEl = document.querySelector('h1, h2, .passage-title, .title');
            if (titleEl) {
                titleText = titleEl.innerText.split('\n')[0].trim();
            }

            // 3. 提取高亮或选中的混合语料内容（同时兼容2个句子+2个短语）
            let finalBatchText = "";
            let selectedText = window.getSelection().toString().trim();

            if (selectedText) {
                finalBatchText = selectedText;
            } else {
                let markedElements = document.querySelectorAll('span[class*="highlight"], span[class*="note"], mark');
                let extractedTexts = [];
                markedElements.forEach((el) => {
                    let text = el.innerText.trim();
                    if (text && !extractedTexts.includes(text)) {
                        extractedTexts.push(text);
                    }
                });
                finalBatchText = extractedTexts.join('\n');
            }

            if (!finalBatchText) {
                alert("💡 提示：没有检测到选中的文本！请先用鼠标抹黑选中你要复盘的重难点句词。");
                return;
            }

            // 🌟 核心更新：已经为你精准对接了最新的“阅读复盘pro max.html”的桌面安全编码路径
               let localReviewPage = "https://moewen0918.github.io/ielts-coach/";
            let finalUrl = `${localReviewPage}?text=${encodeURIComponent(finalBatchText)}&part=${encodeURIComponent(partText)}&title=${encodeURIComponent(titleText)}`;
            window.open(finalUrl, '_blank');
        };
    }
})();
