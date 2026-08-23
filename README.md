[widget.js](https://github.com/user-attachments/files/31344268/widget.js)
/* Виджет-чат «Таткадастр» для сайта (Тильда).
   Дизайн в стиле сайта: #1e3a8a + #f59e0b, Inter, скругления, градиенты.
   Подключение: <script src="https://ВАШ-ДОМЕН/widget.js" defer></script>
   Виджет сам добавляет кнопку "Онлайн-расчёт" в существующий чат-виджет
   сайта (если найдёт .chat-options), либо рисует свою плавающую кнопку. */
(function () {
    'use strict';
    var API = (document.currentScript && document.currentScript.dataset.api) ||
              (location.protocol + '//' + location.host);
    var TG_URL = 'https://t.me/Promo_tatkadastr_bot';
    var MAX_URL = 'https://max.ru/u/f9LHodD0cOLks50Fx6gDLf4gPNiQ0wwB-Ag50HtSJqHQJb2tFOOv48-q4LI';

    var css = [
        '#tkw-window{position:fixed;bottom:100px;right:30px;width:380px;max-width:calc(100vw - 24px);',
        'height:560px;max-height:calc(100vh - 130px);background:#fff;border-radius:20px;z-index:99999;',
        'box-shadow:0 20px 25px -5px rgba(0,0,0,.15),0 10px 10px -5px rgba(0,0,0,.06);display:none;',
        'flex-direction:column;overflow:hidden;font-family:Inter,-apple-system,"Segoe UI",Roboto,sans-serif}',
        '#tkw-window.open{display:flex;animation:tkwPop .25s ease}',
        '@keyframes tkwPop{from{transform:translateY(16px);opacity:0}to{transform:none;opacity:1}}',
        '#tkw-head{background:linear-gradient(135deg,#1e3a8a,#3b82f6);color:#fff;padding:14px 16px;',
        'display:flex;align-items:center;gap:10px}',
        '#tkw-head .tkw-ava{width:38px;height:38px;border-radius:50%;background:#fff;color:#1e3a8a;',
        'display:flex;align-items:center;justify-content:center;font-weight:800;font-size:.9rem}',
        '#tkw-head .tkw-t{flex:1;line-height:1.25}#tkw-head b{font-size:.95rem}',
        '#tkw-head small{opacity:.85;font-size:.75rem;display:block}',
        '#tkw-close{background:rgba(255,255,255,.15);border:none;color:#fff;width:32px;height:32px;',
        'border-radius:50%;cursor:pointer;font-size:1rem;line-height:1}',
        '#tkw-msgs{flex:1;overflow-y:auto;padding:14px 12px;background:#f9fafb;display:flex;',
        'flex-direction:column;gap:8px}',
        '.tkw-m{max-width:88%;padding:9px 13px;border-radius:14px;font-size:.88rem;line-height:1.5;',
        'white-space:pre-wrap;word-wrap:break-word}',
        '.tkw-in{background:#fff;border:1px solid #eef0f3;border-bottom-left-radius:4px;align-self:flex-start;color:#111827}',
        '.tkw-out{background:linear-gradient(135deg,#1e3a8a,#3b82f6);color:#fff;border-bottom-right-radius:4px;align-self:flex-end}',
        '#tkw-kb{padding:6px 10px 10px;background:#f9fafb;display:flex;flex-direction:column;gap:6px}',
        '.tkw-row{display:flex;gap:6px}',
        '.tkw-btn{flex:1;background:#fff;border:1.5px solid #dbe2ec;border-radius:12px;padding:9px 8px;',
        'font-size:.82rem;font-weight:600;color:#1e3a8a;cursor:pointer;font-family:inherit;transition:all .2s}',
        '.tkw-btn:hover{border-color:#f59e0b;background:#fffaf0}',
        '#tkw-input{display:flex;gap:8px;padding:10px 12px;background:#fff;border-top:1px solid #eef0f3}',
        '#tkw-input input{flex:1;border:1.5px solid #dbe2ec;border-radius:24px;padding:10px 14px;',
        'font-size:.9rem;outline:none;font-family:inherit}',
        '#tkw-input input:focus{border-color:#3b82f6}',
        '#tkw-send{background:linear-gradient(135deg,#f59e0b,#fbbf24);border:none;color:#fff;width:42px;',
        'height:42px;border-radius:50%;cursor:pointer;font-size:1rem}',
        '#tkw-mess{display:flex;gap:8px;padding:0 12px 10px;background:#f9fafb}',
        '.tkw-mlink{flex:1;text-align:center;text-decoration:none;font-size:.76rem;font-weight:700;',
        'padding:8px 6px;border-radius:10px;color:#fff}',
        '.tkw-mlink.tg{background:#0088cc}.tkw-mlink.mx{background:#7c3aed}',
        '#tkw-fab{position:fixed;bottom:100px;right:30px;z-index:99998;background:linear-gradient(135deg,#f59e0b,#fbbf24);',
        'color:#fff;border:none;border-radius:50px;padding:14px 22px;font-weight:700;font-size:.9rem;cursor:pointer;',
        'font-family:Inter,sans-serif;box-shadow:0 10px 20px rgba(245,158,11,.35);display:none}',
        '@media(max-width:768px){#tkw-window{right:8px;bottom:80px;width:calc(100vw - 16px);height:70vh}',
        '#tkw-fab{bottom:150px;right:20px;padding:12px 18px;font-size:.82rem}}'
    ].join('');

    var style = document.createElement('style');
    style.textContent = css;
    document.head.appendChild(style);

    var win = document.createElement('div');
    win.id = 'tkw-window';
    win.innerHTML =
        '<div id="tkw-head"><div class="tkw-ava">ТК</div>' +
        '<div class="tkw-t"><b>Таткадастр — онлайн-расчёт</b>' +
        '<small>техплан · техпаспорт · от 2 минут</small></div>' +
        '<button id="tkw-close" aria-label="Закрыть">✕</button></div>' +
        '<div id="tkw-msgs"></div>' +
        '<div id="tkw-mess"><a class="tkw-mlink tg" href="' + TG_URL + '" target="_blank" rel="noopener">Открыть в Telegram</a>' +
        '<a class="tkw-mlink mx" href="' + MAX_URL + '" target="_blank" rel="noopener">Открыть в MAX</a></div>' +
        '<div id="tkw-kb"></div>' +
        '<div id="tkw-input"><input placeholder="Сообщение…" aria-label="Сообщение">' +
        '<button id="tkw-send" aria-label="Отправить">➤</button></div>';
    document.body.appendChild(win);

    var msgs = win.querySelector('#tkw-msgs'),
        kb = win.querySelector('#tkw-kb'),
        inp = win.querySelector('#tkw-input input');

    var session = null, started = false;
    try { session = localStorage.getItem('tkw_session'); } catch (e) {}

    function bubble(text, cls) {
        var d = document.createElement('div');
        d.className = 'tkw-m ' + cls;
        d.textContent = text;
        msgs.appendChild(d);
        msgs.scrollTop = msgs.scrollHeight;
    }
    function renderKb(rows) {
        kb.innerHTML = '';
        (rows || []).forEach(function (r) {
            var row = document.createElement('div');
            row.className = 'tkw-row';
            r.forEach(function (label) {
                var b = document.createElement('button');
                b.className = 'tkw-btn';
                b.textContent = label;
                b.onclick = function () { send(label); };
                row.appendChild(b);
            });
            kb.appendChild(row);
        });
    }
    function send(text) {
        var t = (text !== undefined) ? text : inp.value.trim();
        if (!t) return;
        inp.value = '';
        if (t !== '/start') bubble(t, 'tkw-out');
        fetch(API + '/api/chat', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({ session: session, text: t, source: 'web' })
        }).then(function (r) { return r.json(); }).then(function (data) {
            session = data.session;
            try { localStorage.setItem('tkw_session', session); } catch (e) {}
            var lastButtons = [];
            (data.messages || []).forEach(function (m) {
                bubble(m.text, 'tkw-in');
                if (m.buttons) lastButtons = m.buttons;
            });
            renderKb(lastButtons);
        }).catch(function () {
            bubble('Не получилось связаться с сервером 🙈 Попробуйте ещё раз или напишите нам: +7 937 776-27-11', 'tkw-in');
        });
    }
    inp.addEventListener('keydown', function (e) { if (e.key === 'Enter') send(); });
    win.querySelector('#tkw-send').onclick = function () { send(); };
    win.querySelector('#tkw-close').onclick = closeChat;

    function openChat() {
        win.classList.add('open');
        if (!started) { started = true; send('/start'); }
        try { if (typeof ym === 'function') ym(106426795, 'reachGoal', 'widget_open'); } catch (e) {}
        setTimeout(function () { inp.focus(); }, 200);
    }
    function closeChat() { win.classList.remove('open'); }

    // Кнопка «Онлайн-расчёт»: встраиваем в существующий чат-виджет сайта,
    // иначе — своя плавающая кнопка.
    function mount() {
        var opts = document.querySelector('.chat-options');
        if (opts) {
            var a = document.createElement('a');
            a.href = '#';
            a.className = 'chat-option';
            a.style.background = 'linear-gradient(135deg,#f59e0b,#fbbf24)';
            a.innerHTML = '<i class="fas fa-calculator" aria-hidden="true"></i> Онлайн-расчёт';
            a.setAttribute('data-goal', 'widget_open');
            a.onclick = function (e) { e.preventDefault(); openChat(); };
            opts.insertBefore(a, opts.firstChild);
        } else {
            var fab = document.createElement('button');
            fab.id = 'tkw-fab';
            fab.textContent = '💬 Онлайн-расчёт стоимости';
            fab.style.display = 'block';
            fab.onclick = openChat;
            document.body.appendChild(fab);
        }
    }
    if (document.readyState === 'loading') {
        document.addEventListener('DOMContentLoaded', mount);
    } else { mount(); }
})();
