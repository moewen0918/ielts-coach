# 🎯 IELTS English AI Coach Pro Max | 雅思阅读双引擎 AI 精读复盘工作台

一款专为雅思考生量身定制的**ai智能化、沉浸式阅读精读与错题复盘系统**。

传统复盘往往需要在做题网站、网页翻译、AI 对话框和错题本之间频繁切换。本项目通过 **“前端轻量化容器 + 浏览器自动化脚本”** 的架构，实现了在任意雅思刷题网站上**“鼠标划词/句 -> 一键无缝打包 -> 云端多维解构”**的保姆级复盘体验。

---

## ✨ 核心亮点

*   🚀 **秒级无缝打包投递**：无需繁琐的复制粘贴。配合专属油猴脚本，在阅读外刊或真题时，只需鼠标选中重难点并轻轻一击，语料与上下文便会瞬间“传送”至复盘大盘。
*   🤖 **主流 AI 双引擎驱动**：完美适配 **DeepSeek** 与 **Gemini 3.5/1.5 Flash** 等前沿大模型。双管齐下，既能深度剖析长难句语法结构，又能精准捕获地道近义词替换（Synonym Replacement）。
*   🎨 **极致纯净的备考视觉**：抛弃凌乱的网页干扰，提供专为精读设计的浅色高饱和度排版，让你的每一次复盘都像在阅读精美的高端电子杂志。
*   🔒 **零信任本地隐私架构 (Zero-Knowledge)**：**安全重于一切。** 本项目不设中心化服务器，用户的 API Key 以及所有宝贵的做题足迹、词汇库，全量加密存储于用户自身的浏览器本地（`localStorage`）。GitHub 仅作为静态容器，绝不上传或泄露任何私密密钥。

---

## 🛠️ 技术架构与工作原理

本工具由两部分组成，形成一个完美的本地与云端闭环：
1.  **云端工作台 (`index.html`)**：部署于 GitHub Pages 的核心响应式网页，负责接收语料、调用 AI 接口、渲染精读报告并管理本地错题本。
2.  **本地穿梭机 (`油猴脚本`)**：常驻于用户浏览器内部，动态监听各大雅思模考、刷题网站，实时抓取文本片段并完成 URL 编码投递。
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
