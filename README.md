# nicofull-fixed-titlle
ニコニコフルスクリーン固定タイトル

```
// ==UserScript==
// @name         ニコニコフルスクリーン固定タイトル (メニュー完全連動版)
// @namespace    http://tampermonkey.net
// @version      12.0
// @match        https://*.nicovideo.jp/watch/*
// @grant        none
// ==/UserScript==

(function() {
    'use strict';

    const ID = 'nicofull-fixed-title';
    let hideTimeout = null;

    // タイトル要素を作成する関数
    function createTitleElement() {
        const titleDiv = document.createElement('div');
        titleDiv.id = ID;
        const pageTitle = document.title.replace(/ - ニコニコ動画\$/, '');
        titleDiv.textContent = "タイトル: " + pageTitle;

        titleDiv.style.cssText = `
            position: absolute !important;
            top: 20px !important;
            left: 20px !important;
            z-index: 999999 !important;
            background: rgba(0, 0, 0, 0.85) !important;
            color: #00ffff !important;
            padding: 12px 20px !important;
            border-radius: 6px !important;
            font-size: 18px !important;
            font-weight: bold !important;
            font-family: sans-serif !important;
            pointer-events: none !important;
            border: 2px solid #ffffff !important;
            box-shadow: 0 4px 12px rgba(0,0,0,0.5) !important;
            max-width: 70vw !important;
            white-space: nowrap !important;
            overflow: hidden !important;
            text-overflow: ellipsis !important;
            opacity: 0;
            transition: opacity 0.25s ease-in-out !important;
        `;
        return titleDiv;
    }

    // 表示制御を行うメイン関数 (引数でマウス移動か定期監視かを区別)
    function checkVisibility(isMouseMove = false) {
        const videoElement = document.querySelector('video');
        if (!videoElement) return;

        const target = videoElement.parentElement;
        if (!target) return;

        let titleEl = target.querySelector(`#${ID}`);
        if (!titleEl) {
            titleEl = createTitleElement();
            target.appendChild(titleEl);
        }

        // 見た目が全画面かどうかを計算
        const rect = videoElement.getBoundingClientRect();
        const isFullscreenNow = (rect.width >= window.innerWidth * 0.95 && rect.height >= window.innerHeight * 0.95);

        // 【ガード1】通常画面の時は、何があっても絶対に表示しない
        if (!isFullscreenNow) {
            titleEl.style.opacity = '0';
            if (hideTimeout) clearTimeout(hideTimeout);
            return;
        }

        // 【ガード2】ビデオの親要素内にある「下部メニュー（コントローラー）」を直接探す
        const controller = target.querySelector('[class*="controller"]') ||
                           target.querySelector('[class*="Control"]') ||
                           target.querySelector('[class*="Panel"]');

        if (controller) {
            const computedStyle = window.getComputedStyle(controller);
            const isMenuHidden = computedStyle.opacity === '0' || computedStyle.visibility === 'hidden';

            // ニコニコ側がすでにメニューを消している場合は、タイマーに関わらず即座にタイトルも消す
            if (isMenuHidden) {
                titleEl.style.opacity = '0';
                if (hideTimeout) clearTimeout(hideTimeout);
                return;
            }
        }

        // フルスクリーンかつ、マウスが動いた時のみ表示処理を開始する
        if (isMouseMove) {
            titleEl.style.opacity = '1';

            if (hideTimeout) clearTimeout(hideTimeout);

            // ニコニコのメニューが自動で引っ込むタイミング（約3秒）に合わせてフェードアウト
            hideTimeout = setTimeout(() => {
                if (titleEl) titleEl.style.opacity = '0';
            }, 3000);
        }
    }

    // マウスが動いた時は「表示トリガー（true）」を出して実行
    window.addEventListener('mousemove', () => checkVisibility(true), true);

    // 画面サイズ変更時
    window.addEventListener('resize', () => checkVisibility(false), true);

    // 定期監視（フルスクリーン解除のチェック専用。勝手に表示をオンにしないよう false で実行）
    setInterval(() => checkVisibility(false), 1000);
})();

```
