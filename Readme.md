# 🏴‍☠️ TMO.GG OPRD Full Translator (One Piece Random Defense)

![Version](https://img.shields.io/badge/version-7.3-blue) ![Game](https://img.shields.io/badge/Game-Warcraft%203-green) ![Language](https://img.shields.io/badge/Language-English-orange)

A comprehensive **Tampermonkey Userscript** created by **LiMie** that automatically translates the Korean build helper website [TMO.GG](https://tmo.gg/) for the Warcraft 3 mod *One Piece Random Defense (ORD)* into English.

This tool was specifically developed and optimized for:
👉 **[https://tmo.gg/build-helper/14176](https://tmo.gg/build-helper/14176)**

It bypasses the issue where the website reverts to Korean every time the external bridge tool (`TMO.GG.exe`) updates the game data.

## ✨ Features

* **🔄 Real-Time Aggressive Translation**
  Uses a DOM-walker that checks for changes every **250ms** to instantly translate new text (unit updates, stat changes, program status) as they appear on the screen.
* **🧠 Smart Navigation (v7.3)**
  Navigating unit recipes is now seamless without moving your mouse:
  * **Short Right-Click** or **ESC**: Intelligently detects context.
    1. If you are deep inside a recipe tree, it clicks the **Back Arrow (←)**.
    2. If you are at the top of a unit view, it clicks the **Close Button (X)**.
    3. If a popup/overlay is open, it closes it via the backdrop.
* **🔓 Anti-Copy Bypass**
  Removes the website's restrictions on right-clicking and text selection. You can now freely inspect elements or copy text from the site.
* **📖 Comprehensive RPG Dictionary**
  * **Smart Grammar:** Translates terms like "Chance to", "within Range", "True Dmg" within complex sentences.
  * **Units:** Full translation of **correct One Piece character names** (e.g., Luname, Caesar, Akainu).
  * **Stats:** Translates terms like Slow, Stun, Armor Break (Ab), Magic/Phys Dmg, Mana Regen, Boss/AoE Kill, etc.
* **💡 Tooltips**
  Automatically translates mouse-over information and descriptions.
* **✍️ Credit:**
  Displays a discreet "Translated by LiMie" badge in the bottom right corner.

## 🛠️ Prerequisites

To use this script, you need a modern web browser (Chrome, Edge, Firefox, Opera) and a Userscript manager extension:

* **[Tampermonkey](https://www.tampermonkey.net/)** (Recommended)
* Violentmonkey
* Greasemonkey

## 🚀 Installation Guide

1.  **Install Tampermonkey:** Go to your browser's extension store and install the Tampermonkey extension.
2.  **Create New Script:** Click the Tampermonkey icon in your browser toolbar and select **"Create a new script..."**.
3.  **Clear Editor:** Remove any default code generated in the editor so it is completely empty.
4.  **Paste Code:** Copy the full source code provided below (see section [Source Code](#-source-code-v73)) and paste it into the editor.
5.  **Save:** Press `Ctrl+S` or click **File > Save**.
6.  **Enable:** Go to the "Installed Scripts" tab in the Tampermonkey Dashboard. **Ensure the toggle switch next to the script is turned ON (Green).**
7.  **Activate:** Visit the specific build helper page:
    [https://tmo.gg/build-helper/14176](https://tmo.gg/build-helper/14176)

## ❓ Troubleshooting

* **Error "@match: Could not parse the pattern":** This happens if you accidentally copy Markdown links into the script header. Ensure the URL in the script is plain text: `https://tmo.gg/*`
* **Translation flickers:** This is normal behavior. The website reloads the original Korean text upon every data update from the game. The script detects this change and re-translates it within milliseconds.
* **"Program Disconnected"**: This translates the Korean status "프로그램 미연동". It indicates that your `TMO.GG.exe` bridge program is not currently sending data to the website.

---

## 📜 Source Code (v7.3)

```javascript

// ==UserScript==
// @name         TMO.GG OPRD Full Translator (v8.0 - Visual Upgrade)
// @namespace    http://tampermonkey.net/
// @version      8.0
// @description  Translation + Smart Back + Rank Highlighting + Font Fix.
// @author       LiMie
// @match        https://tmo.gg/*
// @grant        none
// @run-at       document-end
// ==/UserScript==

(function() {
    'use strict';

    console.log("TMO Translator v8.0 by LiMie started...");

    // --- 1. CONFIG & STYLES ---
    const style = document.createElement('style');
    // Erzwinge eine saubere, westliche Schriftart & bessere Lesbarkeit
    style.innerHTML = `
        *, body {
            user-select: text !important;
            -webkit-user-select: text !important;
            cursor: auto !important;
            font-family: 'Segoe UI', 'Roboto', 'Helvetica Neue', Arial, sans-serif !important;
        }
        /* Optional: Scrollbars hübscher machen */
        ::-webkit-scrollbar { width: 8px; }
        ::-webkit-scrollbar-track { background: #1a1a1a; }
        ::-webkit-scrollbar-thumb { background: #444; border-radius: 4px; }
        ::-webkit-scrollbar-thumb:hover { background: #666; }
    `;
    document.head.appendChild(style);

    // --- 2. SMART NAVIGATION (Rechtsklick/ESC Zurück) ---
    let rightClickStartTime = 0;
    document.addEventListener('contextmenu', e => { if (Date.now() - rightClickStartTime < 300) e.preventDefault(); e.stopPropagation(); }, true);
    document.addEventListener('mousedown', e => { if (e.button === 2) rightClickStartTime = Date.now(); });
    document.addEventListener('mouseup', e => { if (e.button === 2 && Date.now() - rightClickStartTime < 300) smartBack(); });
    document.addEventListener('keydown', e => { if (e.key === 'Escape') smartBack(); });

    function smartBack() {
        const btns = Array.from(document.querySelectorAll('button'));
        const close = btns.find(b => {
            const t = (b.innerText || "").toLowerCase();
            const a = (b.getAttribute('aria-label') || "").toLowerCase();
            return t.includes('close') || a.includes('close') || a.includes('back') || a.includes('닫기');
        });
        if (close && close.offsetParent) { close.click(); return; }
        const backdrop = Array.from(document.querySelectorAll('div')).find(d => {
            const s = window.getComputedStyle(d);
            return s.position === 'fixed' && s.zIndex > 100 && s.backgroundColor.includes('rgba');
        });
        if (backdrop) { backdrop.click(); return; }
        if (window.history.length > 1) window.history.back();
    }

    // --- 3. CREDIT (Click to hide) ---
    const addCredit = () => {
        if (document.getElementById('limie-credit')) return;
        const d = document.createElement('div');
        d.id = 'limie-credit';
        d.innerText = 'Translated by LiMie (Click to hide)';
        d.style.cssText = `position:fixed;bottom:10px;right:10px;z-index:99999;background:rgba(0,0,0,0.7);color:#aaa;padding:5px 10px;border-radius:5px;font-size:10px;cursor:pointer;transition:0.3s;`;
        d.onclick = function() { this.style.display = 'none'; };
        d.onmouseover = function() { this.style.color = 'white'; this.style.background = 'rgba(0,0,0,0.9)'; };
        d.onmouseout = function() { this.style.color = '#aaa'; this.style.background = 'rgba(0,0,0,0.7)'; };
        document.body.appendChild(d);
    };
    setTimeout(addCredit, 2000);

    // --- 4. DICTIONARY ---
    const dictionary = [
        // UI & Status
        { k: "프로그램이 정상적으로 연동되었습니다.", v: "Program successfully connected." },
        { k: "프로그램이 실행되지 않았습니다.", v: "Program not running." },
        { k: "클릭하여 프로그램을 실행해주세요.", v: "Click to start program." },
        { k: "프로그램 연동됨", v: "🟢 PROGRAM CONNECTED" },
        { k: "프로그램 미연동", v: "🔴 PROGRAM DISCONNECTED" },
        { k: "라이트모드로 전환", v: "Switch to Light Mode" },
        { k: "다크모드로 전환", v: "Switch to Dark Mode" },
        { k: "조합도우미", v: "Build Helper" },
        { k: "서비스 소개", v: "About Service" },
        { k: "이용약관", v: "Terms of Use" },
        { k: "개인정보처리방침", v: "Privacy Policy" },
        { k: "고객센터", v: "Support Center" },
        { k: "공지사항", v: "Notice" },
        { k: "로그인", v: "Login" },
        { k: "커뮤니티", v: "Community" },
        { k: "복제", v: "Copy Code" },
        { k: "전체화면", v: "Fullscreen" },
        { k: "기본순", v: "Default Sort" },
        { k: "퍼센트 높은순", v: "% High" },
        { k: "퍼센트 낮은순", v: "% Low" },
        { k: "능력치 필터", v: "Stat Filter" },
        { k: "현재 능력치", v: "Current Stats" },
        { k: "풀이감", v: "Max Slow" },
        { k: "풀방깍", v: "Max ArmorBreak" },
        { k: "신", v: "God" },
        { k: "악몽", v: "Nightmare" },
        { k: "자원", v: "Resources" },
        { k: "금화", v: "Gold" },
        { k: "목재", v: "Wood" },
        { k: "특성 포인트", v: "Trait Points" },
        { k: "댓글", v: "Comments" },
        { k: "개인용", v: "Personal" },
        { k: "새로고침", v: "Refresh" },

        // Stats
        { k: "마법 방어력 감소", v: "MagResist Down" },
        { k: "마법 대미지 증가", v: "MagDmg Up" },
        { k: "마법 대미지 증가", v: "MagDmg Up" },
        { k: "마뎀증", v: "MagDmg Up" },
        { k: "폭발형 대미지 증폭", v: "ExplDmg Amp" },
        { k: "폭발뎀증폭", v: "ExplDmg Amp" },
        { k: "폭뎀증", v: "ExplDmg Up" },
        { k: "방어력 무시 대미지", v: "IgnoreDef Dmg" },
        { k: "방어력 무시", v: "IgnoreDef" },
        { k: "방무뎀", v: "IgnoreDef Dmg" },
        { k: "방무딜", v: "IgnoreDef Dmg" },
        { k: "방무", v: "IgnoreDef" },
        { k: "범위 전체 체력 퍼센트 대미지", v: "AoE Max HP % Dmg" },
        { k: "범위 현재 체력 퍼센트 대미지", v: "AoE Curr. HP % Dmg" },
        { k: "전체 체력 퍼센트 대미지", v: "Max HP % Dmg" },
        { k: "현재 체력 퍼센트 대미지", v: "Curr. HP % Dmg" },
        { k: "범위 퍼센트 대미지", v: "AoE % Dmg" },
        { k: "체력 퍼센트 대미지", v: "HP % Dmg" },
        { k: "전퍼스킬", v: "Max HP % Skill" },
        { k: "전퍼", v: "Max %" },
        { k: "현퍼", v: "Curr. %" },
        { k: "잃퍼", v: "Lost %" },
        { k: "모든피해증가", v: "All Dmg Up" },
        { k: "피증", v: "Dmg Up" },
        { k: "공격속도 증가 (단일)", v: "AtkSpd Up (Single)" },
        { k: "공격속도 증가", v: "AtkSpd Up" },
        { k: "공속증가", v: "AtkSpd Up" },
        { k: "공격속도", v: "AtkSpd" },
        { k: "이동속도 감소", v: "Slow" },
        { k: "이동속도", v: "MoveSpd" },
        { k: "발동이감", v: "Proc Slow" },
        { k: "발이감", v: "Proc Slow" },
        { k: "발동방어력 감소", v: "Proc ArmorBreak" },
        { k: "발동공격력 증가", v: "Proc Atk Dmg Up" },
        { k: "발동이동속도 감소", v: "Proc Slow" },
        { k: "단일방어력 감소", v: "Single ArmorBreak" },
        { k: "단일마법 대미지 증가", v: "Single MagDmg Up" },
        { k: "단일아머브레이크", v: "Single ArmorBreak" },
        { k: "단일 방어력 감소", v: "Single ArmorBreak" },
        { k: "중첩방어력 감소", v: "Stack ArmorBreak" },
        { k: "방어력 감소", v: "ArmorBreak" },
        { k: "마나 재생", v: "Mana Reg" },
        { k: "마나젠", v: "Mana Reg" },
        { k: "체력 재생", v: "HP Reg" },
        { k: "공격력 증가", v: "Atk Dmg Up" },
        { k: "아머브레이크", v: "ArmorBreak" },
        { k: "미러미러열매", v: "Mirror Fruit" },
        { k: "보조딜", v: "Support Dmg" },
        { k: "광폭화 잡기", v: "Berserk Kill" },
        { k: "광폭추뎀", v: "Berserk Dmg" },
        { k: "보광잡", v: "Boss/AoE Kill" },
        { k: "보AoE Kill", v: "Boss/AoE Kill" },
        { k: "보스 잡기", v: "Boss Kill" },
        { k: "라인딜", v: "Line Dmg" },
        { k: "스플딜", v: "Splash Dmg" },
        { k: "스플", v: "Splash" },
        { k: "유닛삭제", v: "Unit Delete" },
        { k: "유닛", v: "Unit" },
        { k: "위습생성", v: "Wisp Spawn" },
        { k: "자석", v: "Magnet" },
        { k: "복제", v: "Clone" },
        { k: "강화", v: "Enhance" },
        { k: "중첩", v: "Stack" },
        { k: "탐색", v: "Search" },
        { k: "가능", v: "Possible" },
        { k: "물마", v: "Phys/Mag" },
        { k: "점치기", v: "Fortune Telling" },
        { k: "3단크립O", v: "3-Hit Creep" },
        { k: "흡수", v: "Absorb" },
        { k: "마딜보조", v: "Magic Support" },
        { k: "1시가능", v: "Solo Capable" },
        { k: "1시 가능", v: "Solo Capable" },
        { k: "1인공증", v: "Single DmgBuff" },
        { k: "버프부여", v: "Grant Buff" },
        { k: "발동공", v: "Proc Atk" },
        { k: "마체젠", v: "Mana+HP Reg" },
        { k: "마증", v: "MagDmg Up" },
        { k: "물딜보조", v: "Phys Support" },
        { k: "체마", v: "HP/Mana" },
        { k: "w자석", v: "W-Magnet" },
        { k: "토큰업", v: "Token Up" },
        { k: "폐문", v: "Door Close" },
        { k: "41라이전조합", v: "Craft < 41 Round" },
        { k: "필요", v: "Need" },
        { k: "소모", v: "Cost" },
        { k: "확률로", v: "Chance to" },
        { k: "확률", v: "Chance" },
        { k: "범위 내의", v: "within Range" },
        { k: "범위", v: "Range" },
        { k: "에게", v: "to" },
        { k: "대상", v: "Target" },
        { k: "초당", v: "/sec" },
        { k: "초간", v: "s" },
        { k: "초", v: "s" },
        { k: "동안", v: "for" },
        { k: "추가", v: "Add." },
        { k: "고정 대미지", v: "True Dmg" },
        { k: "마법 대미지", v: "Magic Dmg" },
        { k: "대미지", v: "Damage" },
        { k: "피해량", v: "Damage" },
        { k: "회복", v: "Recover" },
        { k: "증가", v: "Inc" },
        { k: "감소", v: "Dec" },
        { k: "발동", v: "Proc" },
        { k: "획득", v: "Get" },
        { k: "있을수록", v: "the more" },
        { k: "높을수록", v: "higher" },
        { k: "없음", v: "None" },
        { k: "일반", v: "Normal" },
        { k: "지형무시", v: "Ignore Terrain" },
        { k: "공중이동", v: "Air Move" },
        { k: "바다이동", v: "Sea Move" },
        { k: "이동", v: "Move" },
        { k: "과 중복 안됨", v: "does not stack" },
        { k: "중복 안됨", v: "No Stack" },
        { k: "사용 시", v: "On Use:" },
        { k: "공격 시", v: "On Hit:" },
        { k: "처형", v: "Execute" },
        { k: "패널티", v: "Penalty" },
        { k: "변신", v: "Transform" },
        { k: "소환", v: "Summon" },
        { k: "적", v: "Enemy" },
        { k: "보스", v: "Boss" },
        { k: "스토리", v: "Story" },
        { k: "광폭화", v: "Berserk" },
        { k: "삭제", v: "Delete" },
        { k: "목업", v: "Wood Up" },
        { k: "특성", v: "Trait" },

        // Short Stats
        { k: "이감", v: "Slow" },
        { k: "방깍", v: "Ab" },
        { k: "발깍", v: "Proc Ab" },
        { k: "공증", v: "DmgBuff" },
        { k: "공속", v: "AtkSpd" },
        { k: "스턴", v: "Stun" },
        { k: "암브", v: "ArmorBreak" },
        { k: "마젠", v: "Mana Reg" },
        { k: "체젠", v: "HP Reg" },
        { k: "단일", v: "Single" },
        { k: "끝딜", v: "Finisher" },
        { k: "범퍼", v: "AoE Buff" },
        { k: "광보잡", v: "AoE Boss" },
        { k: "보잡", v: "Boss Kill" },
        { k: "광잡", v: "AoE Kill" },
        { k: "블링크", v: "Blink" },
        { k: "물딜", v: "Phys" },
        { k: "마딜", v: "Magic" },
        { k: "물뎀", v: "Phys" },
        { k: "마뎀", v: "Magic" },
        { k: "깍", v: "Ab" },
        { k: "바제스", v: "Burgess" },

        // Ranks
        { k: "안흔함", v: "Uncommon" },
        { k: "특별함", v: "Special" }, { k: "특별", v: "Special" },
        { k: "희귀함", v: "Rare" }, { k: "희귀", v: "Rare" },
        { k: "전설", v: "Legendary" },
        { k: "히든", v: "Hidden" },
        { k: "변화된", v: "Changed" }, { k: "변화", v: "Changed" },
        { k: "초월", v: "Transcendence" },
        { k: "불멸", v: "Immortal" },
        { k: "영원", v: "Eternity" },
        { k: "제한됨", v: "Limited" },
        { k: "랜덤 전용", v: "Random Only" }, { k: "랜덤", v: "Random" },
        { k: "신비", v: "Mystery" },
        { k: "흔함", v: "Common" },
        { k: "기타", v: "Other" },
        { k: "왜곡됨", v: "Distorted" },
        { k: "해적선", v: "Pirate Ship" },

        // Units
        { k: "루나메", v: "Luname" },
        { k: "시저", v: "Caesar" },
        { k: "쵸파 두뇌강화", v: "Chopper Brain Pt" },
        { k: "쵸파 가드 포인트", v: "Chopper Guard Pt" },
        { k: "쵸파 혼 포인트", v: "Chopper Horn Pt" },
        { k: "쵸파 중량강화", v: "Chopper Heavy Pt" },
        { k: "쵸파 럼블볼", v: "Chopper Rumble" },
        { k: "쵸파 유력강화", v: "Chopper Arm Pt" },
        { k: "쵸파 탱크", v: "Chopper Tank" },
        { k: "쵸파", v: "Chopper" },
        { k: "루피 기어세컨드", v: "Luffy Gear 2" },
        { k: "루피 기어서드", v: "Luffy Gear 3" },
        { k: "니카(루초)", v: "Nika (Luffy)" },
        { k: "니카(뱀초)", v: "Nika (Snake)" },
        { k: "루피", v: "Luffy" },
        { k: "조로 귀기", v: "Zoro Asura" },
        { k: "조로 염왕", v: "Zoro Enma" },
        { k: "조로", v: "Zoro" },
        { k: "나미 크리마 텍트", v: "Nami Clima-Tact" },
        { k: "나미", v: "Nami" },
        { k: "저격왕 우솝", v: "Sniper King" },
        { k: "우솝 화염탄", v: "Usopp Fire" },
        { k: "우솝", v: "Usopp" },
        { k: "상디 검은다리", v: "Sanji Black Leg" },
        { k: "상디 디아블잠브", v: "Sanji Diable" },
        { k: "상디 제르마", v: "Sanji Germa" },
        { k: "상디", v: "Sanji" },
        { k: "프랑키", v: "Franky" },
        { k: "브룩", v: "Brook" },
        { k: "로빈 오하라", v: "Robin Ohara" },
        { k: "로빈", v: "Robin" },
        { k: "징베", v: "Jinbe" },
        { k: "해군 총병", v: "Marine Gunner" },
        { k: "해군 칼병", v: "Marine Sword" },
        { k: "거프", v: "Garp" },
        { k: "센고쿠", v: "Sengoku" },
        { k: "아카이누", v: "Akainu" },
        { k: "키자루", v: "Kizaru" },
        { k: "아오키지", v: "Aokiji" },
        { k: "후지토라", v: "Fujitora" },
        { k: "료쿠규", v: "Ryokugyu" },
        { k: "제파", v: "Zephyr" },
        { k: "코비", v: "Koby" },
        { k: "타시기", v: "Tashigi" },
        { k: "스모커", v: "Smoker" },
        { k: "히바리", v: "Hibari" },
        { k: "모몬가", v: "Momonga" },
        { k: "베르고", v: "Vergo" },
        { k: "루치", v: "Lucci" },
        { k: "로브 루치", v: "Rob Lucci" },
        { k: "스튜시", v: "Stussy" },
        { k: "카쿠", v: "Kaku" },
        { k: "블루노", v: "Blueno" },
        { k: "후쿠로", v: "Fukuro" },
        { k: "마젤란", v: "Magellan" },
        { k: "시류", v: "Shiryu" },
        { k: "흰수염", v: "Whitebeard" },
        { k: "검은수염", v: "Blackbeard" },
        { k: "티치", v: "Teach" },
        { k: "에이스 2번대대장", v: "Ace 2nd Div" },
        { k: "에이스", v: "Ace" },
        { k: "마르코", v: "Marco" },
        { k: "죠즈", v: "Jozu" },
        { k: "비스타", v: "Vista" },
        { k: "스쿼드", v: "Squard" },
        { k: "이조", v: "Izo" },
        { k: "샹크스", v: "Shanks" },
        { k: "레일리", v: "Rayleigh" },
        { k: "벤 베크만", v: "Ben Beckman" },
        { k: "로져", v: "Roger" },
        { k: "스코퍼가반", v: "Gaban" },
        { k: "불릿", v: "Bullet" },
        { k: "레드필드", v: "Redfield" },
        { k: "시키", v: "Shiki" },
        { k: "버기 마기탄", v: "Buggy Muggy" },
        { k: "버기", v: "Buggy" },
        { k: "크로커다일", v: "Crocodile" },
        { k: "미호크", v: "Mihawk" },
        { k: "도플라밍고", v: "Doflamingo" },
        { k: "쿠마", v: "Kuma" },
        { k: "모리아", v: "Moria" },
        { k: "겟코모리아", v: "Moria" },
        { k: "핸콕", v: "Hancock" },
        { k: "징베", v: "Jinbe" },
        { k: "로우", v: "Law" },
        { k: "키드", v: "Kid" },
        { k: "캡틴 키드", v: "Capt. Kid" },
        { k: "갱 벳지", v: "Bege" },
        { k: "바질 호킨스", v: "Hawkins" },
        { k: "바질호킨스", v: "Hawkins" },
        { k: "드레이크", v: "Drake" },
        { k: "아푸", v: "Apoo" },
        { k: "보니", v: "Bonney" },
        { k: "킬러", v: "Killer" },
        { k: "울티", v: "Ulti" },
        { k: "블랙마리아", v: "Black Maria" },
        { k: "카이도", v: "Kaido" },
        { k: "킹", v: "King" },
        { k: "퀸", v: "Queen" },
        { k: "잭", v: "Jack" },
        { k: "빅맘", v: "Big Mom" },
        { k: "카타쿠리", v: "Katakuri" },
        { k: "크래커", v: "Cracker" },
        { k: "샬롯 브륄레", v: "Brulee" },
        { k: "페로나", v: "Perona" },
        { k: "바르톨로메오", v: "Bartolomeo" },
        { k: "캐럿", v: "Carrot" },
        { k: "야마토", v: "Yamato" },
        { k: "오뎅", v: "Oden" },
        { k: "킨에몬", v: "Kinemon" },
        { k: "라이조", v: "Raizo" },
        { k: "슈가", v: "Sugar" },
        { k: "베이비 5", v: "Baby 5" },
        { k: "반 더 데켄", v: "Vander Decken" },
        { k: "호디 존스", v: "Hody Jones" },
        { k: "시라호시", v: "Shirahoshi" },
        { k: "아론", v: "Arlong" },
        { k: "하찌", v: "Hachi" },
        { k: "갓 에넬", v: "God Enel" },
        { k: "에넬", v: "Enel" },
        { k: "와이퍼", v: "Wyper" },
        { k: "카르가라", v: "Calgara" },
        { k: "이완코브", v: "Ivankov" },
        { k: "사보", v: "Sabo" },
        { k: "코알라", v: "Koala" },
        { k: "드래곤", v: "Dragon" },
        { k: "이나즈마 혁명군", v: "Inazuma Rev" },
        { k: "이나즈마", v: "Inazuma" },
        { k: "베포", v: "Bepo" },
        { k: "제프", v: "Zeff" },
        { k: "비비", v: "Vivi" },
        { k: "카벤딧슈", v: "Cavendish" },
        { k: "라분", v: "Laboon" },
        { k: "류마", v: "Ryuma" },
        { k: "바제스", v: "Burgess" },
        { k: "시노부", v: "Shinobu" },
        { k: "레이쥬", v: "Reiju" },
        { k: "네코", v: "Nekomamushi" },
        { k: "이누", v: "Inuarashi" },
        { k: "뱀초", v: "Snake Man" },
        { k: "우타", v: "Uta" },
        { k: "알비다", v: "Alvida" },
        { k: "캡틴 크로", v: "Capt. Kuro" },
        { k: "돈 클리크", v: "Don Krieg" },
        { k: "아인", v: "Ain" },
        { k: "피셔타이거", v: "Fisher Tiger" },
        { k: "키쿠", v: "Kiku" },
        { k: "봉쿠레", v: "Bon Clay" },
        { k: "센토마루", v: "Sentomaru" },
        { k: "오즈", v: "Oars" },
        { k: "레베카", v: "Rebecca" },
        { k: "도킹 5", v: "Docking 5" },
        { k: "압살롬", v: "Absalom" },
        { k: "챠카", v: "Chaka" },
        { k: "헤르메포", v: "Helmeppo" },
        { k: "타시기", v: "Tashigi" },
        { k: "레드포스호", v: "Red Force" },
        { k: "모비딕호", v: "Moby Dick" },
        { k: "발라티에", v: "Baratie" },
        { k: "방주맥심", v: "Ark Maxim" },
        { k: "써니호", v: "Sunny Go" },
        { k: "고대의 배", v: "Ancient Ship" },
        { k: "나루토 선인모드", v: "Naruto Sage" },
        { k: "고죠사토루", v: "Gojo Satoru" },
        { k: "미나토", v: "Minato" },
        { k: "료우기 시키", v: "Ryougi Shiki" },
        { k: "히그마", v: "Higuma" },
        { k: "부릉냐", v: "Bronya" },
        { k: "아냐", v: "Anya" },
        { k: "유카리", v: "Yukari" },
        { k: "요우무", v: "Youmu" },
        { k: "죠타로", v: "Jotaro" },
        { k: "바쿠야", v: "Byakuya" },
        { k: "네즈코", v: "Nezuko" },
        { k: "타츠마키", v: "Tatsumaki" },
        { k: "샌즈", v: "Sans" },
        { k: "메구밍", v: "Megumin" },
        { k: "센토 이스즈", v: "Sento Isuzu" },
        { k: "야가미 라이토", v: "Light Yagami" },
        { k: "옌", v: "Yen" },
        { k: "요츠바", v: "Yotsuba" },
        { k: "이사야마 요미", v: "Isayama Yomi" },
        { k: "이치의 율자", v: "Herrscher" },
        { k: "전투펭귄", v: "Battle Penguin" },
        { k: "카미조 토우마", v: "Kamijou Touma" },
        { k: "쿠로사키 이치고", v: "Ichigo" },
        { k: "페이몬", v: "Paimon" },
        { k: "하네카와 츠바사", v: "Hanekawa" },
        { k: "미니스트로맨", v: "Mini Stroman" },
        { k: "토큰", v: "Token" },
        { k: "좀비", v: "Zombie" },
        { k: "위습", v: "Wisp" }
    ];

    // ** AUTO SORT DICTIONARY BY LENGTH (Longest first) **
    dictionary.sort((a, b) => b.k.length - a.k.length);

    function translateText(text) {
        if (!text) return text;
        let newText = text;
        for (let i = 0; i < dictionary.length; i++) {
            if (newText.includes(dictionary[i].k)) {
                // Global replacement
                newText = newText.split(dictionary[i].k).join(dictionary[i].v);
            }
        }
        return newText;
    }

    function traverseAndTranslate(node) {
        if (node.nodeType === 3) { // Text
            const original = node.nodeValue;
            if (original && original.trim() !== "") {
                const translated = translateText(original);
                if (original !== translated) {
                    node.nodeValue = translated;
                }
            }
        } else if (node.nodeType === 1) { // Element
            // Color Highlighting for Ranks
            if (node.childNodes.length === 1 && node.childNodes[0].nodeType === 3) {
                const txt = node.innerText;
                if (txt === 'Eternity') { node.style.color = '#00FFFF'; node.style.fontWeight = 'bold'; }
                else if (txt === 'Immortal') { node.style.color = '#FFD700'; node.style.fontWeight = 'bold'; }
                else if (txt === 'Transcendence') { node.style.color = '#FF69B4'; node.style.fontWeight = 'bold'; }
                else if (txt === 'Limited') { node.style.color = '#FFA500'; node.style.fontWeight = 'bold'; }
            }

            // Tooltips
            if (node.hasAttribute('data-tooltip-content')) {
                const originalTip = node.getAttribute('data-tooltip-content');
                const translatedTip = translateText(originalTip);
                if (originalTip !== translatedTip) {
                    node.setAttribute('data-tooltip-content', translatedTip);
                }
            }
            // ARIA Labels
            if (node.hasAttribute('aria-label')) {
                const label = node.getAttribute('aria-label');
                const translatedLabel = translateText(label);
                if (label !== translatedLabel) {
                     node.setAttribute('aria-label', translatedLabel);
                }
            }
            // Recursion
            if (node.tagName !== 'SCRIPT' && node.tagName !== 'STYLE' && node.tagName !== 'NOSCRIPT') {
                for (let i = 0; i < node.childNodes.length; i++) {
                    traverseAndTranslate(node.childNodes[i]);
                }
            }
        }
    }

    function updateTitle() {
        if (document.title === "개인용") document.title = "TMO.GG | Personal";
    }

    // Run loop
    setInterval(() => {
        if (document.body) {
            traverseAndTranslate(document.body);
            updateTitle();
        }
    }, 250);

})();
