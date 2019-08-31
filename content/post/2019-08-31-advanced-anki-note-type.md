---
title: "Advanced Anki Note Type"
date: 2019-08-31T20:59:18+09:00
draft: false
---

Quick article to present you the note type I'm always using to learn Japanese words/verbs with examples.

## Quick setup

Just [download this Anki deck](/assets/2019-08-31-advanced-anki-note-type/Export.apkg),
delete the "Export" deck and freely use the new note type for your card.

Note type should follow the pattern "Japanese (opt reversed)" followed by some numbers.

## Fields

All fields are using Arial 20 font:

1. **Primary** (Exp: 食事) (sortable field)
2. **Reading** (Exp: しょくじ)
3. **Meaning1** (Exp: meal, dinner)
4. **Meaning2** (Exp: diet)
5. **Context** (Exp: する)
6. **Exp1** (Exp: Your robot will prepare meals, clean, wash dishes, and perform other household tasks.)
7. **Exp1 JP** (Exp: あなたのロボットは食事の 支度[したく]、 掃除[そうじ]、 皿洗い[さらあら] その他の 家事[かじ]ができるでしょう。)
8. **Exp2** (Exp: She went on a reducing diet.)
9. **Exp2 JP** (Exp: 彼女はやせるために 食事[しょくじ]を 制限[せいげん]した。)

## Card Types

### EN  -> JP

Front template:

```html
{{#Meaning1}} <!-- Conditional display if at least Meaning1 is existing -->
<p>{{#Meaning2}}<b>1. </b>{{/Meaning2}}{{Meaning1}}</p>
{{#Meaning2}}
<p><b>2. </b>{{Meaning2}}</p>
{{/Meaning2}}
{{#Context}}({{Context}}){{/Context}}

<br>
<br>

{{#Exp1}}
<p class="extra">{{Exp1}}<p>
{{/Exp1}}

{{#Exp2}}
<p class="extra">{{Exp2}}<p>
{{/Exp2}}
{{/Meaning1}}
```

<img src="/assets/2019-08-31-advanced-anki-note-type/en-front.png" alt="en-front" style="width: 500px; border: 1px solid black"/>

Back template:

```html
<p>{{#Meaning2}}<b>1. </b>{{/Meaning2}}{{Meaning1}}</p>
{{#Meaning2}}
<p><b>2. </b>{{Meaning2}}</p>
{{/Meaning2}}
{{#Context}}({{Context}}){{/Context}}

<hr id=answer>

<p class="jp">{{Primary}}</p>
<p class="jp">{{Reading}}</p>

{{#Exp1}}
<hr>
<p class="extra">{{Exp1}}<p>
<p class="extra jp">{{furigana:Exp1 JP}}<p>
{{/Exp1}}

{{#Exp2}}
<p class="extra">{{Exp2}}<p>
<p class="extra jp">{{furigana:Exp2 JP}}<p>
{{/Exp2}}
```

<img src="/assets/2019-08-31-advanced-anki-note-type/en-back.png" alt="en-back" style="width: 500px; border: 1px solid black"/>

### KJ -> all

Front template:

```html
{{#Reading}}
<p class="jp">{{Primary}}</p>
{{#Context}}({{Context}}){{/Context}}
{{/Reading}}
```

<img src="/assets/2019-08-31-advanced-anki-note-type/kj-front.png" alt="kj-front" style="width: 500px; border: 1px solid black"/>

Back template:

```html
{{FrontSide}}

<hr id=answer>

<p class="jp">{{Reading}}</p>

<p>{{#Meaning2}}<b>1. </b>{{/Meaning2}}{{Meaning1}}</p>
{{#Meaning2}}
<p><b>2. </b>{{Meaning2}}</p>
{{/Meaning2}}

{{#Exp1}}
<hr>
<p class="extra">{{Exp1}}<p>
<p class="extra jp">{{furigana:Exp1 JP}}<p>
{{/Exp1}}

{{#Exp2}}
<p class="extra">{{Exp2}}<p>
<p class="extra jp">{{furigana:Exp2 JP}}<p>
{{/Exp2}}
```

<img src="/assets/2019-08-31-advanced-anki-note-type/kj-back.png" alt="kj-back" style="width: 500px; border: 1px solid black"/>

### Exp1

Front template:

```html
<!-- Conditional display if at least Exp1 JP is existing -->
{{#Exp1 JP}}

<p class="jp">{{furigana::Exp1 JP}}</p>

<b><p class="jp">{{Primary}}</p></b>

{{#Reading}}
<p class="jp extra">{{hint:Reading}}</p>
{{/Reading}}

{{#Context}}  ({{Context}})<br/>  {{/Context}}

<hr>

<p>{{#Meaning2}}<b>1. </b>{{/Meaning2}}{{hint:Meaning1}}</p>

{{#Meaning2}}
<p><b>2. </b>{{hint:Meaning2}}</p>
{{/Meaning2}}

{{/Exp1 JP}}
```

<img src="/assets/2019-08-31-advanced-anki-note-type/exp1-front.png" alt="exp1-front" style="width: 500px; border: 1px solid black"/>

Back template:

```html
<p class="jp">{{furigana::Exp1 JP}}</p>
<p class="detail">{{Exp1}}</p>

{{#Exp2}}
<hr>
<p class="jp">{{furigana:Exp2 JP}}<p>
<p class="detail">{{furigana::Exp2}}<p>
{{/Exp2}}

<hr>

<p class="jp">{{Primary}}</p>
{{#Reading}}
<p class="jp">{{Reading}}</p>
{{/Reading}}

{{#Context}} <p>({{Context}})</p>  {{/Context}}

<p>{{#Meaning2}}<b>1. </b>{{/Meaning2}}{{Meaning1}}</p>

{{#Meaning2}}
<p><b>2. </b>{{Meaning2}}</p>
{{/Meaning2}}
```

<img src="/assets/2019-08-31-advanced-anki-note-type/exp1-back.png" alt="exp1-back" style="width: 500px; border: 1px solid black"/>

## Card style (for all card types)

```css
.card {
 font-family: arial;
 font-size: 20px;
 text-align: center;
 color: black;
 background-color: white;
}

.extra { font-size: 11px; }

.jp { font-size: 30px; }

.extra.jp { font-size: 20px; }
.win .jp { font-family: "MS Mincho", "ＭＳ 明朝"; }
.mac .jp { font-family: "Hiragino Mincho Pro", "ヒラギノ明朝 Pro"; }
.linux .jp { font-family: "Kochi Mincho", "東風明朝"; }
.mobile .jp { font-family: "Hiragino Mincho ProN"; }
```
