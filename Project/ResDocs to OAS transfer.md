---
작성일: 2026-05-10
cssclasses:
  - wide
  - table-wide
tags:
  - ResDocs2OAS
---

## Kanban 카드

```dataviewjs
const tag = "#" + dv.current().tags?.[0]
const kanbanPath = "Kanban Board.md"
const kanbanFile = app.vault.getAbstractFileByPath(kanbanPath)
const content = await app.vault.read(kanbanFile)
const lines = content.split('\n')

let cards = [], currentCard = null, currentSection = ""

for (const line of lines) {
  const sec = line.match(/^## (.+)/)
  if (sec) { currentSection = sec[1].trim(); continue }

  const card = line.match(/^- \[([ x])\] ####\s*(.+)/)
  if (card) {
    if (currentCard) cards.push(currentCard)
    currentCard = { done: card[1]==='x', title: card[2].trim(), section: currentSection, hasTag: false, items: [], date: null }
    continue
  }

  if (currentCard) {
    if (line.includes(tag)) currentCard.hasTag = true
    const d = line.match(/@\{(\d{4}-\d{2}-\d{2})\}/)
    if (d) currentCard.date = d[1]
    const item = line.match(/^\t+- \[([ x])\] (.+)/)
    if (item) currentCard.items.push({ done: item[1]==='x', text: item[2].trim() })
  }
}
if (currentCard) cards.push(currentCard)

const matched = cards.filter(c => c.hasTag)
if (!matched.length) { dv.paragraph("관련 카드 없음"); return }

const colors = { 'Back Log': '#6b7280', 'To Do': '#3b82f6', 'In Progress': '#f59e0b', 'Done': '#10b981' }
const kanbanLink = `obsidian://open?vault=${encodeURIComponent(app.vault.getName())}&file=${encodeURIComponent(kanbanPath)}`

for (const card of matched) {
  const color = colors[card.section] ?? '#6b7280'
  const done = card.items.filter(i => i.done).length
  const total = card.items.length
  const pct = total ? Math.round(done / total * 100) : 0

  const wrap = dv.el('div', '', { attr: { style: 'border:1px solid var(--background-modifier-border);border-radius:8px;padding:14px;margin-bottom:12px;' }})

  const head = wrap.createEl('div', { attr: { style: 'display:flex;align-items:center;gap:8px;margin-bottom:10px;' }})
  head.createEl('span', { text: card.section, attr: { style: `background:${color}22;color:${color};border-radius:4px;padding:2px 8px;font-size:12px;font-weight:600;` }})
  head.createEl('a', { text: card.title, href: kanbanLink, attr: { style: 'font-size:15px;font-weight:600;color:var(--text-normal);text-decoration:none;' }})
  if (card.date) head.createEl('span', { text: '📅 ' + card.date, attr: { style: 'margin-left:auto;font-size:12px;color:var(--text-muted);' }})

  if (total > 0) {
    const prog = wrap.createEl('div', { attr: { style: 'margin-bottom:8px;' }})
    const bar = prog.createEl('div', { attr: { style: 'background:var(--background-modifier-border);border-radius:4px;height:6px;overflow:hidden;margin-bottom:4px;' }})
    bar.createEl('div', { attr: { style: `background:${color};width:${pct}%;height:100%;transition:width 0.3s;` }})
    prog.createEl('span', { text: `${done} / ${total} 완료`, attr: { style: 'font-size:11px;color:var(--text-muted);' }})
  }

  if (card.items.length > 0) {
    const ul = wrap.createEl('ul', { attr: { style: 'margin:8px 0 0;padding-left:0;list-style:none;' }})
    for (const item of card.items) {
      ul.createEl('li', { text: (item.done ? '✅ ' : '⬜ ') + item.text, attr: { style: `font-size:13px;color:${item.done ? 'var(--text-muted)' : 'var(--text-normal)'};text-decoration:${item.done ? 'line-through' : 'none'};margin-bottom:4px;` }})
    }
  }
}
```