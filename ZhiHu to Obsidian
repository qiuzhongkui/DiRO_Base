// ==UserScript==
// @name         ZhiHu to Obsidian
// @version      2.2.0
// @description  抓取知乎问题/专栏，实时预览，图片标注提示手动搬运
// @author       digital rose
// @match        https://www.zhihu.com/question/*
// @match        https://zhuanlan.zhihu.com/p/*
// @grant        GM_xmlhttpRequest
// @grant        GM_getValue
// @grant        GM_setValue
// @connect      localhost
// @connect      127.0.0.1
// @downloadURL https://update.greasyfork.org/scripts/563370/%E7%9F%A5%E4%B9%8E%E5%9B%9E%E7%AD%94%E5%AF%BC%E5%87%BA%E5%88%B0%20Obsidian.user.js
// @updateURL https://update.greasyfork.org/scripts/563370/%E7%9F%A5%E4%B9%8E%E5%9B%9E%E7%AD%94%E5%AF%BC%E5%87%BA%E5%88%B0%20Obsidian.meta.js
// ==/UserScript==

(function() {
    'use strict';

    // 默认配置
    const DEFAULT_CONFIG = {
        obsidianHost: 'http://localhost:27123',
        obsidianApiKey: '',
        storagePath: '知乎收藏',
        imageFolder: 'attachments'
    };

    function getConfig() {
        return {
            obsidianHost: GM_getValue('obsidianHost', DEFAULT_CONFIG.obsidianHost),
            obsidianApiKey: GM_getValue('obsidianApiKey', DEFAULT_CONFIG.obsidianApiKey),
            storagePath: GM_getValue('storagePath', DEFAULT_CONFIG.storagePath),
            imageFolder: GM_getValue('imageFolder', DEFAULT_CONFIG.imageFolder)
        };
    }

    function saveConfig(config) {
        GM_setValue('obsidianHost', config.obsidianHost);
        GM_setValue('obsidianApiKey', config.obsidianApiKey);
        GM_setValue('storagePath', config.storagePath);
        GM_setValue('imageFolder', config.imageFolder);
    }

    function escapeHtml(text) {
        const div = document.createElement('div');
        div.textContent = text;
        return div.innerHTML;
    }

    function createFloatingWindow() {
        const container = document.createElement('div');
        container.id = 'zhihu-obsidian-sync';
        container.innerHTML = `
            <style>
                #zhihu-obsidian-sync {
                    position: fixed;
                    bottom: 20px;
                    right: 20px;
                    z-index: 10000;
                    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
                }

                .sync-toggle-btn {
                    width: 56px;
                    height: 56px;
                    border-radius: 50%;
                    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
                    border: none;
                    cursor: pointer;
                    box-shadow: 0 4px 12px rgba(0,0,0,0.15);
                    display: flex;
                    align-items: center;
                    justify-content: center;
                    transition: all 0.3s ease;
                    color: white;
                    font-size: 24px;
                }

                .sync-toggle-btn:hover {
                    transform: scale(1.1);
                    box-shadow: 0 6px 16px rgba(0,0,0,0.2);
                }

                .sync-panel {
                    display: none;
                    position: absolute;
                    bottom: 70px;
                    right: 0;
                    width: 360px;
                    background: white;
                    border-radius: 12px;
                    box-shadow: 0 8px 32px rgba(0,0,0,0.12);
                    overflow: hidden;
                }

                .sync-panel.show {
                    display: block;
                    animation: slideUp 0.3s ease;
                }

                @keyframes slideUp {
                    from {
                        opacity: 0;
                        transform: translateY(10px);
                    }
                    to {
                        opacity: 1;
                        transform: translateY(0);
                    }
                }

                .sync-header {
                    padding: 16px 20px;
                    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
                    color: white;
                    font-weight: 600;
                    font-size: 16px;
                }

                .sync-content {
                    padding: 20px;
                }

                .sync-main {
                    margin-bottom: 16px;
                }

                .sync-btn {
                    width: 100%;
                    padding: 12px;
                    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
                    color: white;
                    border: none;
                    border-radius: 8px;
                    cursor: pointer;
                    font-size: 14px;
                    font-weight: 500;
                    transition: all 0.2s ease;
                }

                .sync-btn:hover {
                    transform: translateY(-2px);
                    box-shadow: 0 4px 12px rgba(102, 126, 234, 0.4);
                }

                .sync-btn:disabled {
                    opacity: 0.6;
                    cursor: not-allowed;
                    transform: none;
                }

                .sync-settings-toggle {
                    padding: 12px;
                    background: #f7f7f7;
                    border: none;
                    border-radius: 8px;
                    cursor: pointer;
                    width: 100%;
                    text-align: left;
                    font-size: 14px;
                    color: #333;
                    display: flex;
                    justify-content: space-between;
                    align-items: center;
                    transition: background 0.2s ease;
                }

                .sync-settings-toggle:hover {
                    background: #efefef;
                }

                .sync-settings-toggle::after {
                    content: '▼';
                    font-size: 12px;
                    transition: transform 0.3s ease;
                }

                .sync-settings-toggle.active::after {
                    transform: rotate(180deg);
                }

                .sync-settings {
                    max-height: 0;
                    overflow: hidden;
                    transition: max-height 0.3s ease;
                }

                .sync-settings.show {
                    max-height: 600px;
                    margin-top: 16px;
                }

                .sync-form-group {
                    margin-bottom: 16px;
                }

                .sync-form-group label {
                    display: block;
                    margin-bottom: 6px;
                    font-size: 13px;
                    color: #666;
                    font-weight: 500;
                }

                .sync-form-group input {
                    width: 100%;
                    padding: 10px 12px;
                    border: 1px solid #e0e0e0;
                    border-radius: 6px;
                    font-size: 14px;
                    box-sizing: border-box;
                    transition: border-color 0.2s ease;
                }

                .sync-form-group input:focus {
                    outline: none;
                    border-color: #667eea;
                }

                .sync-hint {
                    font-size: 12px;
                    color: #999;
                    margin-top: 4px;
                    line-height: 1.4;
                }

                .sync-hint a {
                    color: #667eea;
                    text-decoration: none;
                }

                .sync-hint a:hover {
                    text-decoration: underline;
                }

                .sync-test-btn {
                    width: 100%;
                    padding: 10px;
                    background: white;
                    color: #667eea;
                    border: 1px solid #667eea;
                    border-radius: 6px;
                    cursor: pointer;
                    font-size: 14px;
                    transition: all 0.3s ease;
                    font-weight: 500;
                }

                .sync-test-btn:hover {
                    background: #667eea;
                    color: white;
                }

                .sync-test-btn.success {
                    background: #28a745;
                    color: white;
                    border-color: #28a745;
                }

                .sync-test-btn.success:hover {
                    background: #218838;
                    border-color: #218838;
                }

                .sync-save-btn {
                    width: 100%;
                    padding: 10px;
                    background: #667eea;
                    color: white;
                    border: none;
                    border-radius: 6px;
                    cursor: pointer;
                    font-size: 14px;
                    transition: all 0.2s ease;
                }

                .sync-save-btn:hover {
                    background: #5568d3;
                }

                .sync-status {
                    margin-top: 12px;
                    padding: 10px;
                    border-radius: 6px;
                    font-size: 13px;
                    text-align: center;
                }

                .sync-status.success {
                    background: #d4edda;
                    color: #155724;
                }

                .sync-status.error {
                    background: #f8d7da;
                    color: #721c24;
                }

                .sync-status.info {
                    background: #d1ecf1;
                    color: #0c5460;
                }

                .sync-modal-overlay {
                    display: none;
                    position: fixed;
                    top: 0;
                    left: 0;
                    width: 100%;
                    height: 100%;
                    background: rgba(0, 0, 0, 0.5);
                    z-index: 10001;
                    justify-content: center;
                    align-items: center;
                }

                .sync-modal-overlay.show {
                    display: flex;
                }

                .sync-modal {
                    background: white;
                    border-radius: 12px;
                    width: 90%;
                    max-width: 700px;
                    max-height: 80vh;
                    display: flex;
                    flex-direction: column;
                    box-shadow: 0 20px 60px rgba(0,0,0,0.3);
                    animation: modalSlideIn 0.3s ease;
                }

                @keyframes modalSlideIn {
                    from {
                        opacity: 0;
                        transform: translateY(-20px);
                    }
                    to {
                        opacity: 1;
                        transform: translateY(0);
                    }
                }

                .sync-modal-header {
                    padding: 16px 20px;
                    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
                    color: white;
                    font-weight: 600;
                    font-size: 16px;
                    border-radius: 12px 12px 0 0;
                    display: flex;
                    justify-content: space-between;
                    align-items: center;
                }

                .sync-modal-close {
                    background: none;
                    border: none;
                    color: white;
                    font-size: 24px;
                    cursor: pointer;
                    padding: 0;
                    line-height: 1;
                    opacity: 0.8;
                }

                .sync-modal-close:hover {
                    opacity: 1;
                }

                .sync-modal-body {
                    padding: 20px;
                    overflow-y: auto;
                    flex: 1;
                }

                .sync-table {
                    width: 100%;
                    border-collapse: collapse;
                    font-size: 14px;
                }

                .sync-table th,
                .sync-table td {
                    padding: 12px;
                    text-align: left;
                    border-bottom: 1px solid #eee;
                }

                .sync-table th {
                    background: #f8f9fa;
                    font-weight: 600;
                    color: #333;
                    position: sticky;
                    top: 0;
                }

                .sync-table tr:hover {
                    background: #f8f9fa;
                }

                .sync-table td:first-child,
                .sync-table th:first-child {
                    width: 40px;
                    text-align: center;
                }

                .sync-table td:nth-child(2),
                .sync-table th:nth-child(2) {
                    width: 70px;
                    text-align: center;
                }

                .sync-table td:nth-child(3),
                .sync-table th:nth-child(3) {
                    width: 100px;
                }

                .sync-table .answer-summary {
                    color: #666;
                    font-size: 13px;
                    line-height: 1.4;
                    max-width: 350px;
                    overflow: hidden;
                    text-overflow: ellipsis;
                    white-space: nowrap;
                }

                .sync-table input[type="checkbox"] {
                    width: 18px;
                    height: 18px;
                    cursor: pointer;
                }

                .sync-modal-footer {
                    padding: 16px 20px;
                    border-top: 1px solid #eee;
                    display: flex;
                    justify-content: space-between;
                    align-items: center;
                    gap: 12px;
                }

                .sync-select-all {
                    display: flex;
                    align-items: center;
                    gap: 8px;
                    font-size: 14px;
                    color: #666;
                    cursor: pointer;
                }

                .sync-confirm-btn {
                    padding: 10px 24px;
                    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
                    color: white;
                    border: none;
                    border-radius: 8px;
                    cursor: pointer;
                    font-size: 14px;
                    font-weight: 500;
                    transition: all 0.2s ease;
                }

                .sync-confirm-btn:hover {
                    transform: translateY(-2px);
                    box-shadow: 0 4px 12px rgba(102, 126, 234, 0.4);
                }

                .sync-confirm-btn:disabled {
                    opacity: 0.6;
                    cursor: not-allowed;
                    transform: none;
                }

                .sync-loading {
                    text-align: center;
                    padding: 40px;
                    color: #666;
                }

                .sync-empty {
                    text-align: center;
                    padding: 40px;
                    color: #999;
                }
            </style>

            <button class="sync-toggle-btn" id="syncToggleBtn">📝</button>

            <div class="sync-panel" id="syncPanel">
                <div class="sync-header">知乎 → Obsidian</div>
                <div class="sync-content">
                    <div id="syncPreviewMode">
                        <div class="sync-form-group" style="margin-bottom: 12px;">
                            <label style="font-size: 12px; color: #999;">当前焦点回答预览</label>
                        </div>
                        <div id="syncPreviewBox" style="background: #f8f9fa; border-radius: 8px; padding: 12px; margin-bottom: 12px; max-height: 200px; overflow-y: auto; border: 1px solid #e0e0e0;">
                            <div style="text-align: center; color: #999; font-size: 13px; padding: 20px 0;">滚动到要导出的回答，点击"抓取"按钮</div>
                        </div>
                        <button class="sync-btn" id="syncNowBtn">抓取当前回答</button>
                        <button class="sync-settings-toggle" id="settingsToggle" style="margin-top: 12px;">
                            ⚙️ 设置
                        </button>
                    </div>

                    <div class="sync-settings" id="syncSettings">
                        <div class="sync-form-group">
                            <label>Obsidian 连接地址</label>
                            <input type="text" id="obsidianHost" placeholder="http://localhost:27123">
                            <div class="sync-hint">需要安装 <a href="https://github.com/coddingtonbear/obsidian-local-rest-api" target="_blank">Local REST API</a> 插件</div>
                        </div>

                        <div class="sync-form-group">
                            <label>API Key</label>
                            <input type="password" id="obsidianApiKey" placeholder="在 Obsidian 插件设置中获取">
                        </div>

                        <div class="sync-form-group">
                            <button class="sync-test-btn" id="testConnectionBtn">测试连接</button>
                        </div>

                        <div class="sync-form-group">
                            <label>存储路径</label>
                            <input type="text" id="storagePath" placeholder="知乎收藏">
                        </div>

                        <div class="sync-form-group">
                            <label>图片文件夹</label>
                            <input type="text" id="imageFolder" placeholder="attachments">
                        </div>
                    </div>

                    <div class="sync-status" id="syncStatus" style="display: none;"></div>
                </div>
            </div>
        `;

        document.body.appendChild(container);

        // 绑定事件
        bindEvents();

        // 加载配置
        loadConfigToUI();

        // 启动预览更新
        startPreviewUpdater();
    }

    // 绑定事件
    function bindEvents() {
        const toggleBtn = document.getElementById('syncToggleBtn');
        const panel = document.getElementById('syncPanel');
        const settingsToggle = document.getElementById('settingsToggle');
        const settings = document.getElementById('syncSettings');
        const syncNowBtn = document.getElementById('syncNowBtn');
        const testConnectionBtn = document.getElementById('testConnectionBtn');

        const obsidianHost = document.getElementById('obsidianHost');
        const obsidianApiKey = document.getElementById('obsidianApiKey');
        const storagePath = document.getElementById('storagePath');
        const imageFolder = document.getElementById('imageFolder');

        toggleBtn.addEventListener('click', () => {
            panel.classList.toggle('show');
        });

        settingsToggle.addEventListener('click', () => {
            settingsToggle.classList.toggle('active');
            settings.classList.toggle('show');
        });

        syncNowBtn.addEventListener('click', syncCurrentAnswer);

        testConnectionBtn.addEventListener('click', testConnection);

        obsidianHost.addEventListener('input', autoSaveSettings);
        obsidianApiKey.addEventListener('input', autoSaveSettings);
        storagePath.addEventListener('input', autoSaveSettings);
        imageFolder.addEventListener('input', autoSaveSettings);
    }

    function autoSaveSettings() {
        const config = {
            obsidianHost: document.getElementById('obsidianHost').value.trim(),
            obsidianApiKey: document.getElementById('obsidianApiKey').value.trim(),
            storagePath: document.getElementById('storagePath').value.trim(),
            imageFolder: document.getElementById('imageFolder').value.trim()
        };

        saveConfig(config);
    }

    // 获取当前焦点的回答
    function getFocusedAnswer() {
        const url = window.location.href;

        // 判断页面类型
        const isQuestion = url.includes('/question/');
        const isArticle = url.includes('zhuanlan.zhihu.com/p/');

        if (!isQuestion && !isArticle) {
            return null;
        }

        let questionId, questionTitle, focusedItem, contentEl, authorEl, voteBtn, timeEl, answerLink, answerId;

        if (isQuestion) {
            // ===== 问题页面逻辑 =====
            const questionMatch = url.match(/question\/(\d+)/);
            questionId = questionMatch ? questionMatch[1] : '';
            questionTitle = document.querySelector('.QuestionHeader-title')?.textContent.trim() || '未知问题';

            // 获取视口中最接近顶部的回答
            const answerItems = document.querySelectorAll('.AnswerItem');
            let minDistance = Infinity;

            answerItems.forEach(item => {
                const rect = item.getBoundingClientRect();
                if (rect.top >= 0 && rect.top < minDistance) {
                    minDistance = rect.top;
                    focusedItem = item;
                }
            });

            if (!focusedItem) {
                return null;
            }

            authorEl = focusedItem.querySelector('.AuthorInfo-name');
            contentEl = focusedItem.querySelector('.RichContent-inner');
            voteBtn = focusedItem.querySelector('button[aria-label^="赞同"]');
            timeEl = focusedItem.querySelector('.ContentItem-time');
            answerLink = focusedItem.querySelector('a[href*="/answer/"]');

            if (answerLink) {
                const match = answerLink.href.match(/answer\/(\d+)/);
                answerId = match ? match[1] : '';
            }

        } else if (isArticle) {
            // ===== 专栏页面逻辑 =====
            const articleMatch = url.match(/zhuanlan\.zhihu\.com\/p\/(\d+)/);
            questionId = articleMatch ? articleMatch[1] : '';

            // 获取文章标题 - 尝试多个选择器
            const titleSelectors = [
                '.Post-Title',
                'h1.Post-Title',
                '.ArticleItem-title',
                'h1[class*="Title"]',
                'h1',
                '[data-testid="post_title"]',
                '.title'
            ];

            for (let selector of titleSelectors) {
                const titleEl = document.querySelector(selector);
                if (titleEl) {
                    questionTitle = titleEl.textContent.trim();
                    if (questionTitle) break;
                }
            }

            if (!questionTitle) {
                questionTitle = '未知文章';
            }

            // 获取文章内容 - 尝试多个选择器
            const contentSelectors = [
                '.Post-RichTextContainer',
                '.post-content',
                '[data-testid="post_content"]',
                '[itemprop="articleBody"]',
                '.RichContent-inner',
                '.RichContent',
                '.css-i1dkk0',
                'article',
                '[role="article"]'
            ];

            for (let selector of contentSelectors) {
                const elem = document.querySelector(selector);
                if (elem && elem.textContent.trim().length > 50) {
                    contentEl = elem;
                    break;
                }
            }

            if (!contentEl) {
                return null;
            }

            focusedItem = contentEl;

            // 获取作者名称 - 尝试多个选择器
            const authorSelectors = [
                '.AuthorInfo-name',
                '[data-testid="post_author_name"]',
                '.Post-Author',
                '.author-name',
                'a[data-testid="creator_name"]',
                '.creator-name',
                '[class*="author"] span'
            ];

            for (let selector of authorSelectors) {
                const elem = document.querySelector(selector);
                if (elem) {
                    authorEl = elem;
                    break;
                }
            }

            // 获取发布时间 - 尝试多个选择器
            const timeSelectors = [
                '.ContentItem-time',
                '[data-testid="post_date"]',
                '.Post-time',
                'time',
                '[class*="time"]',
                '.publish-time'
            ];

            for (let selector of timeSelectors) {
                const elem = document.querySelector(selector);
                if (elem) {
                    timeEl = elem;
                    break;
                }
            }

            // 获取赞同按钮 - 尝试多个选择器
            const voteSelectors = [
                'button[aria-label^="赞同"]',
                '[data-testid="like_button"]',
                'button[aria-label*="同"]',
                '[class*="vote"]',
                'button[class*="like"]'
            ];

            for (let selector of voteSelectors) {
                const elem = document.querySelector(selector);
                if (elem) {
                    voteBtn = elem;
                    break;
                }
            }

            answerId = questionId;
        }

        if (!contentEl) {
            return null;
        }

        const author = authorEl?.textContent.trim() || '匿名用户';
        const content = contentEl.innerHTML || '';
        const textContent = contentEl.textContent.trim() || '';

        if (!content || textContent.length < 10) {
            return null;
        }

        let upvotes = '0';
        if (voteBtn) {
            const ariaLabel = voteBtn.getAttribute('aria-label');
            if (ariaLabel) {
                const match = ariaLabel.match(/赞同\s*([\d.,]+\s*[万kKwW]?)/);
                if (match) {
                    upvotes = match[1].replace(/\s/g, '');
                }
            }
            // 备选方案：从按钮文本提取
            if (upvotes === '0') {
                const btnText = voteBtn.textContent.trim();
                const match = btnText.match(/([\d.,]+\s*[万kKwW]?)/);
                if (match) {
                    upvotes = match[1].replace(/\s/g, '');
                }
            }
        }

        const publishTime = timeEl?.textContent.trim() || '';

        // 构建最终的URL
        let finalUrl;
        if (isQuestion && answerId) {
            finalUrl = `https://www.zhihu.com/question/${questionId}/answer/${answerId}`;
        } else {
            finalUrl = url;
        }

        return {
            author,
            upvotes,
            content,
            answerId: answerId || 'unknown',
            questionId,
            questionTitle,
            publishTime,
            url: finalUrl,
            textContent
        };
    }

    // 启动预览更新器
    function startPreviewUpdater() {
        setInterval(() => {
            updatePreview();
        }, 500);
    }

    // 更新预览
    function updatePreview() {
        const previewBox = document.getElementById('syncPreviewBox');
        const answer = getFocusedAnswer();

        if (!answer || !answer.content) {
            previewBox.innerHTML = '<div style="text-align: center; color: #999; font-size: 13px; padding: 20px 0;">打开知乎问题或专栏文章，点击"抓取"按钮</div>';
            return;
        }

        const summary = answer.textContent.length > 150 ? answer.textContent.substring(0, 150) + '...' : answer.textContent;

        previewBox.innerHTML = `
            <div style="font-size: 12px; color: #666;">
                <div style="margin-bottom: 8px;"><strong>作者：</strong> ${escapeHtml(answer.author)}</div>
                <div style="margin-bottom: 8px;"><strong>赞同：</strong> ${answer.upvotes}</div>
                <div style="margin-bottom: 8px;"><strong>摘要：</strong></div>
                <div style="background: white; padding: 8px; border-radius: 4px; max-height: 100px; overflow-y: auto; line-height: 1.4; color: #999; font-size: 11px;">
                    ${escapeHtml(summary)}
                </div>
            </div>
        `;
    }

    // 同步当前焦点回答
    async function syncCurrentAnswer() {
        const answer = getFocusedAnswer();

        if (!answer || !answer.content) {
            showStatus('未检测到有效的回答内容', 'error');
            return;
        }

        const syncNowBtn = document.getElementById('syncNowBtn');
        syncNowBtn.disabled = true;
        syncNowBtn.textContent = '抓取中...';

        const config = getConfig();

        try {
            await syncSingleAnswer(answer, config);

            syncNowBtn.disabled = false;
            syncNowBtn.textContent = '抓取当前回答';
            showStatus('✓ 成功抓取回答到 Obsidian', 'success');
        } catch (error) {
            console.error('抓取失败:', error);
            syncNowBtn.disabled = false;
            syncNowBtn.textContent = '抓取当前回答';
            showStatus('✗ 抓取失败: ' + error.message, 'error');
        }
    }

    async function syncSingleAnswer(answer, config) {
        const questionTitle = answer.questionTitle;
        const questionId = answer.questionId;
        const date = new Date().toISOString().split('T')[0];

        const { markdown, imageMap } = htmlToMarkdown(answer.content);

        let finalMarkdown = markdown;

        // 处理图片：标注提示手动搬运（知乎图片加密，无法自动提取）
        for (const img of imageMap) {
            const imageNote = `\n> [!warning] 📷 手动搬运图片\n> 原图链接：${img.src}\n> 原图说明：${img.alt}\n`;
            finalMarkdown = finalMarkdown.replace(
                `[[IMAGE_PLACEHOLDER_${img.index}]]`,
                imageNote
            );
        }

        const answerContent = finalMarkdown;

        const fileName = `${questionTitle.replace(/[/\\?%*:|"<>]/g, '-')}.md`;
        const filePath = `${config.storagePath}/${fileName}`;

        const existingContent = await checkFileExists(config, filePath);

        if (existingContent) {
            const updatedContent = existingContent + '\n---\n\n' + answerContent;
            await saveToObsidian(config, filePath, updatedContent);
        } else {
            const fullDocument = `---
title: ${questionTitle}
author: ${answer.author}
source: ${answer.url}
date: ${date}
---

${answerContent}
`;
            await saveToObsidian(config, filePath, fullDocument);
        }
    }

    function loadConfigToUI() {
        const config = getConfig();
        document.getElementById('obsidianHost').value = config.obsidianHost;
        document.getElementById('obsidianApiKey').value = config.obsidianApiKey;
        document.getElementById('storagePath').value = config.storagePath;
        document.getElementById('imageFolder').value = config.imageFolder;
    }

    function testConnection() {
        const host = document.getElementById('obsidianHost').value.trim();
        const apiKey = document.getElementById('obsidianApiKey').value.trim();
        const testBtn = document.getElementById('testConnectionBtn');

        if (!apiKey) {
            showStatus('✗ 请先配置 API Key', 'error');
            return;
        }

        testBtn.textContent = '测试中...';
        testBtn.disabled = true;
        testBtn.classList.remove('success');

        GM_xmlhttpRequest({
            method: 'GET',
            url: host,
            headers: {
                'Authorization': `Bearer ${apiKey}`
            },
            timeout: 5000,
            onload: function(response) {
                if (response.status === 200) {
                    try {
                        const data = JSON.parse(response.responseText);
                        if (data.status === 'OK' && data.authenticated === true) {
                            showStatus('✓ 连接成功，API Key 已验证', 'success');
                            testBtn.classList.add('success');
                            testBtn.textContent = '✓ 连接成功';
                        } else if (data.status === 'OK' && data.authenticated === false) {
                            showStatus('✗ API Key 无效或未认证', 'error');
                            testBtn.classList.remove('success');
                            testBtn.textContent = '测试连接';
                        } else {
                            showStatus('✗ 响应格式异常', 'error');
                            testBtn.classList.remove('success');
                            testBtn.textContent = '测试连接';
                        }
                    } catch (e) {
                        showStatus('✗ 解析响应失败', 'error');
                        testBtn.classList.remove('success');
                        testBtn.textContent = '测试连接';
                    }
                } else if (response.status === 401 || response.status === 403) {
                    showStatus('✗ API Key 错误或无权限', 'error');
                    testBtn.classList.remove('success');
                    testBtn.textContent = '测试连接';
                } else {
                    showStatus('✗ 连接失败: ' + response.status, 'error');
                    testBtn.classList.remove('success');
                    testBtn.textContent = '测试连接';
                }
                testBtn.disabled = false;
            },
            onerror: function() {
                showStatus('✗ 连接失败，请检查 Obsidian Local REST API 插件是否已启动', 'error');
                testBtn.classList.remove('success');
                testBtn.textContent = '测试连接';
                testBtn.disabled = false;
            },
            ontimeout: function() {
                showStatus('✗ 连接超时', 'error');
                testBtn.classList.remove('success');
                testBtn.textContent = '测试连接';
                testBtn.disabled = false;
            }
        });
    }

    function showStatus(message, type) {
        const status = document.getElementById('syncStatus');
        status.textContent = message;
        status.className = 'sync-status ' + type;
        status.style.display = 'block';

        setTimeout(() => {
            status.style.display = 'none';
        }, 3000);
    }

// 将 HTML 转换为 Markdown
    function htmlToMarkdown(html) {
        const temp = document.createElement('div');
        temp.innerHTML = html;

        // 处理图片 - 替换为占位符
        const images = temp.querySelectorAll('img');
        const imageMap = [];
        images.forEach((img, index) => {
            const src = img.getAttribute('data-original') || img.src;
            const alt = img.alt || `image-${index}`;
            imageMap.push({ src, alt, index });
            img.replaceWith(`[[IMAGE_PLACEHOLDER_${index}]]`);
        });

        // 处理链接 - 只保留文本，忽视链接语法
        temp.querySelectorAll('a').forEach(a => {
            const text = a.textContent;
            a.replaceWith(text);
        });

        // 处理标题
        for (let i = 1; i <= 6; i++) {
            temp.querySelectorAll(`h${i}`).forEach(h => {
                h.replaceWith(`${'#'.repeat(i)} ${h.textContent}\n\n`);
            });
        }

        // 处理粗体
        temp.querySelectorAll('b, strong').forEach(b => {
            b.replaceWith(`**${b.textContent}**`);
        });

        // 处理斜体
        temp.querySelectorAll('i, em').forEach(i => {
            i.replaceWith(`*${i.textContent}*`);
        });

        // 处理代码块
        temp.querySelectorAll('pre code').forEach(code => {
            code.parentElement.replaceWith(`\n\`\`\`\n${code.textContent}\n\`\`\`\n`);
        });

        // 处理行内代码
        temp.querySelectorAll('code').forEach(code => {
            if (code.parentElement.tagName.toLowerCase() !== 'pre') {
                code.replaceWith(`\`${code.textContent}\``);
            }
        });

        // 处理列表
        temp.querySelectorAll('ul li').forEach(li => {
            li.replaceWith(`- ${li.textContent}\n`);
        });

        temp.querySelectorAll('ol li').forEach((li, index) => {
            li.replaceWith(`${index + 1}. ${li.textContent}\n`);
        });

        // 处理段落
        temp.querySelectorAll('p').forEach(p => {
            p.replaceWith(`${p.textContent}\n\n`);
        });

        // 处理换行
        temp.querySelectorAll('br').forEach(br => {
            br.replaceWith('\n');
        });

        let markdown = temp.textContent;

        // 清理多余空行
        markdown = markdown.replace(/\n{3,}/g, '\n\n');

        return { markdown, imageMap };
    }

    // 下载图片并上传到 Obsidian
async function checkFileExists(config, filePath) {
        return new Promise((resolve) => {
            GM_xmlhttpRequest({
                method: 'GET',
                url: `${config.obsidianHost}/vault/${encodeURIComponent(filePath)}`,
                headers: {
                    'Authorization': `Bearer ${config.obsidianApiKey}`
                },
                onload: function(response) {
                    if (response.status === 200) {
                        resolve(response.responseText);
                    } else {
                        resolve(null);
                    }
                },
                onerror: function() {
                    resolve(null);
                }
            });
        });
    }

    async function saveToObsidian(config, filePath, content) {
        return new Promise((resolve, reject) => {
            const headers = {
                'Content-Type': 'text/markdown'
            };
            if (config.obsidianApiKey) {
                headers['Authorization'] = `Bearer ${config.obsidianApiKey}`;
            }

            GM_xmlhttpRequest({
                method: 'PUT',
                url: `${config.obsidianHost}/vault/${encodeURIComponent(filePath)}`,
                headers: headers,
                data: content,
                onload: function(response) {
                    if (response.status === 200 || response.status === 204) {
                        resolve();
                    } else {
                        reject(new Error('保存失败: ' + response.status));
                    }
                },
                onerror: function() {
                    reject(new Error('保存失败，请检查连接'));
                }
            });
        });
    }

    // 初始化
    if (document.readyState === 'loading') {
        document.addEventListener('DOMContentLoaded', createFloatingWindow);
    } else {
        createFloatingWindow();
    }
})();
