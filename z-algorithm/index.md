# Z 函数及其求法（扩展 KMP）


## Z 函数

（以下默认字符串下标从`0`开始）。

在 KMP 字符串匹配算法中，使用了$\pi$来表示一个字符串的最长公共前后缀，现在我们介绍一个与公共前缀有关的函数，Z 函数。我们用$z_i$表示一个字符串与它以$i$为起点的后缀的最长公共前缀（LCP）。此外，定义$z_0$为$0$。例如，字符串`abcabca`的$z$值分别为$0, 0, 0, 4, 0, 0, 1$。

## Z 函数的求法

根据 Z 函数的定义，我们可以枚举待求字符串的的每一个后缀，从头开始尝试匹配，得到 Z 函数的值。这样计算的时间复杂度是$O(n^2)$。

参考代码（`std::string`，字符串下标从`0`开始）：
```cpp
void GetZFunc(const std::string str)
{
    int len = str.length();
    for (int i = 1; i < len; i++)
        while (i + z[i] < len && str[z[i]] == str[i + z[i]]) 
            z[i]++;
    return;
}
```

## Z 算法（在$O(n)$时间内求出 Z 函数）

类似 KMP 字符串匹配中求$\pi$和 Manacher 算法中求最长回文时的优化思路，我们来想一想能不能利用已有信息减少暴力匹配的计算次数。

为了方便解释，我们把与$s_0 \dots s_{z_i}$匹配的子串$s_i \dots s_{i + z_i - 1}$称为“匹配子串”。我们计算 Z 的时候记录右端点最靠右的“匹配子串”$s_l \dots s_r$，这时如果$i \le r$，那么$s_i \dots s_r$一定与$s_{i - l} \dots s_{r - l}$相同。此时$z_{i - l}$与$z_i$是相同的。但要注意，如果$z_{i - l} \ge r - i + 1$，那么不能保证$r$后面的字符和$r - l$后面的字符匹配，需要暴力尝试。

如果$i \ge r$，由于没有已知信息可以利用，也需要暴力尝试。

以上过程可以用下面的图表示：
```tex
\rlap{
    \underbrace{
        \phantom{ s_0 \dots s_{i - l} \dots s_i \dots s_{r - l} \dots}
        }_\text{same}
    }
s_0 \dots s_{i - l} \dots
    \overbrace{ s_l \dots s_i \dots s_{r - l} \dots s_r
}^\text{same} \dots
```
{{< raw >}}
<svg xmlns="http://www.w3.org/2000/svg" width="34.369ex" height="7.544ex" viewBox="0 -1738.8 15191 3334.5" xmlns:xlink="http://www.w3.org/1999/xlink" aria-hidden="true"><defs><path id="MJX-2-TEX-S4-E152" d="M-24 327L-18 333H-1Q11 333 15 333T22 329T27 322T35 308T54 284Q115 203 225 162T441 120Q454 120 457 117T460 95V60V28Q460 8 457 4T442 0Q355 0 260 36Q75 118 -16 278L-24 292V327Z"></path><path id="MJX-2-TEX-S4-E153" d="M-10 60V95Q-10 113 -7 116T9 120Q151 120 250 171T396 284Q404 293 412 305T424 324T431 331Q433 333 451 333H468L474 327V292L466 278Q375 118 190 36Q95 0 8 0Q-5 0 -7 3T-10 24V60Z"></path><path id="MJX-2-TEX-S4-E151" d="M-10 60Q-10 104 -10 111T-5 118Q-1 120 10 120Q96 120 190 84Q375 2 466 -158L474 -172V-207L468 -213H451H447Q437 -213 434 -213T428 -209T423 -202T414 -187T396 -163Q331 -82 224 -41T9 0Q-4 0 -7 3T-10 25V60Z"></path><path id="MJX-2-TEX-S4-E150" d="M-18 -213L-24 -207V-172L-16 -158Q75 2 260 84Q334 113 415 119Q418 119 427 119T440 120Q454 120 457 117T460 98V60V25Q460 7 457 4T441 0Q308 0 193 -55T25 -205Q21 -211 18 -212T-1 -213H-18Z"></path><path id="MJX-2-TEX-S4-E154" d="M-10 0V120H410V0H-10Z"></path><path id="MJX-2-TEX-N-73" d="M295 316Q295 356 268 385T190 414Q154 414 128 401Q98 382 98 349Q97 344 98 336T114 312T157 287Q175 282 201 278T245 269T277 256Q294 248 310 236T342 195T359 133Q359 71 321 31T198 -10H190Q138 -10 94 26L86 19L77 10Q71 4 65 -1L54 -11H46H42Q39 -11 33 -5V74V132Q33 153 35 157T45 162H54Q66 162 70 158T75 146T82 119T101 77Q136 26 198 26Q295 26 295 104Q295 133 277 151Q257 175 194 187T111 210Q75 227 54 256T33 318Q33 357 50 384T93 424T143 442T187 447H198Q238 447 268 432L283 424L292 431Q302 440 314 448H322H326Q329 448 335 442V310L329 304H301Q295 310 295 316Z"></path><path id="MJX-2-TEX-N-61" d="M137 305T115 305T78 320T63 359Q63 394 97 421T218 448Q291 448 336 416T396 340Q401 326 401 309T402 194V124Q402 76 407 58T428 40Q443 40 448 56T453 109V145H493V106Q492 66 490 59Q481 29 455 12T400 -6T353 12T329 54V58L327 55Q325 52 322 49T314 40T302 29T287 17T269 6T247 -2T221 -8T190 -11Q130 -11 82 20T34 107Q34 128 41 147T68 188T116 225T194 253T304 268H318V290Q318 324 312 340Q290 411 215 411Q197 411 181 410T156 406T148 403Q170 388 170 359Q170 334 154 320ZM126 106Q126 75 150 51T209 26Q247 26 276 49T315 109Q317 116 318 175Q318 233 317 233Q309 233 296 232T251 223T193 203T147 166T126 106Z"></path><path id="MJX-2-TEX-N-6D" d="M41 46H55Q94 46 102 60V68Q102 77 102 91T102 122T103 161T103 203Q103 234 103 269T102 328V351Q99 370 88 376T43 385H25V408Q25 431 27 431L37 432Q47 433 65 434T102 436Q119 437 138 438T167 441T178 442H181V402Q181 364 182 364T187 369T199 384T218 402T247 421T285 437Q305 442 336 442Q351 442 364 440T387 434T406 426T421 417T432 406T441 395T448 384T452 374T455 366L457 361L460 365Q463 369 466 373T475 384T488 397T503 410T523 422T546 432T572 439T603 442Q729 442 740 329Q741 322 741 190V104Q741 66 743 59T754 49Q775 46 803 46H819V0H811L788 1Q764 2 737 2T699 3Q596 3 587 0H579V46H595Q656 46 656 62Q657 64 657 200Q656 335 655 343Q649 371 635 385T611 402T585 404Q540 404 506 370Q479 343 472 315T464 232V168V108Q464 78 465 68T468 55T477 49Q498 46 526 46H542V0H534L510 1Q487 2 460 2T422 3Q319 3 310 0H302V46H318Q379 46 379 62Q380 64 380 200Q379 335 378 343Q372 371 358 385T334 402T308 404Q263 404 229 370Q202 343 195 315T187 232V168V108Q187 78 188 68T191 55T200 49Q221 46 249 46H265V0H257L234 1Q210 2 183 2T145 3Q42 3 33 0H25V46H41Z"></path><path id="MJX-2-TEX-N-65" d="M28 218Q28 273 48 318T98 391T163 433T229 448Q282 448 320 430T378 380T406 316T415 245Q415 238 408 231H126V216Q126 68 226 36Q246 30 270 30Q312 30 342 62Q359 79 369 104L379 128Q382 131 395 131H398Q415 131 415 121Q415 117 412 108Q393 53 349 21T250 -11Q155 -11 92 58T28 218ZM333 275Q322 403 238 411H236Q228 411 220 410T195 402T166 381T143 340T127 274V267H333V275Z"></path><path id="MJX-2-TEX-I-1D460" d="M131 289Q131 321 147 354T203 415T300 442Q362 442 390 415T419 355Q419 323 402 308T364 292Q351 292 340 300T328 326Q328 342 337 354T354 372T367 378Q368 378 368 379Q368 382 361 388T336 399T297 405Q249 405 227 379T204 326Q204 301 223 291T278 274T330 259Q396 230 396 163Q396 135 385 107T352 51T289 7T195 -10Q118 -10 86 19T53 87Q53 126 74 143T118 160Q133 160 146 151T160 120Q160 94 142 76T111 58Q109 57 108 57T107 55Q108 52 115 47T146 34T201 27Q237 27 263 38T301 66T318 97T323 122Q323 150 302 164T254 181T195 196T148 231Q131 256 131 289Z"></path><path id="MJX-2-TEX-N-30" d="M96 585Q152 666 249 666Q297 666 345 640T423 548Q460 465 460 320Q460 165 417 83Q397 41 362 16T301 -15T250 -22Q224 -22 198 -16T137 16T82 83Q39 165 39 320Q39 494 96 585ZM321 597Q291 629 250 629Q208 629 178 597Q153 571 145 525T137 333Q137 175 145 125T181 46Q209 16 250 16Q290 16 318 46Q347 76 354 130T362 333Q362 478 354 524T321 597Z"></path><path id="MJX-2-TEX-N-2026" d="M78 60Q78 84 95 102T138 120Q162 120 180 104T199 61Q199 36 182 18T139 0T96 17T78 60ZM525 60Q525 84 542 102T585 120Q609 120 627 104T646 61Q646 36 629 18T586 0T543 17T525 60ZM972 60Q972 84 989 102T1032 120Q1056 120 1074 104T1093 61Q1093 36 1076 18T1033 0T990 17T972 60Z"></path><path id="MJX-2-TEX-I-1D456" d="M184 600Q184 624 203 642T247 661Q265 661 277 649T290 619Q290 596 270 577T226 557Q211 557 198 567T184 600ZM21 287Q21 295 30 318T54 369T98 420T158 442Q197 442 223 419T250 357Q250 340 236 301T196 196T154 83Q149 61 149 51Q149 26 166 26Q175 26 185 29T208 43T235 78T260 137Q263 149 265 151T282 153Q302 153 302 143Q302 135 293 112T268 61T223 11T161 -11Q129 -11 102 10T74 74Q74 91 79 106T122 220Q160 321 166 341T173 380Q173 404 156 404H154Q124 404 99 371T61 287Q60 286 59 284T58 281T56 279T53 278T49 278T41 278H27Q21 284 21 287Z"></path><path id="MJX-2-TEX-N-2212" d="M84 237T84 250T98 270H679Q694 262 694 250T679 230H98Q84 237 84 250Z"></path><path id="MJX-2-TEX-I-1D459" d="M117 59Q117 26 142 26Q179 26 205 131Q211 151 215 152Q217 153 225 153H229Q238 153 241 153T246 151T248 144Q247 138 245 128T234 90T214 43T183 6T137 -11Q101 -11 70 11T38 85Q38 97 39 102L104 360Q167 615 167 623Q167 626 166 628T162 632T157 634T149 635T141 636T132 637T122 637Q112 637 109 637T101 638T95 641T94 647Q94 649 96 661Q101 680 107 682T179 688Q194 689 213 690T243 693T254 694Q266 694 266 686Q266 675 193 386T118 83Q118 81 118 75T117 65V59Z"></path><path id="MJX-2-TEX-I-1D45F" d="M21 287Q22 290 23 295T28 317T38 348T53 381T73 411T99 433T132 442Q161 442 183 430T214 408T225 388Q227 382 228 382T236 389Q284 441 347 441H350Q398 441 422 400Q430 381 430 363Q430 333 417 315T391 292T366 288Q346 288 334 299T322 328Q322 376 378 392Q356 405 342 405Q286 405 239 331Q229 315 224 298T190 165Q156 25 151 16Q138 -11 108 -11Q95 -11 87 -5T76 7T74 17Q74 30 114 189T154 366Q154 405 128 405Q107 405 92 377T68 316T57 280Q55 278 41 278H27Q21 284 21 287Z"></path></defs><g stroke="currentColor" fill="currentColor" stroke-width="0" transform="matrix(1 0 0 -1 0 0)"><g data-mml-node="math"><g data-mml-node="TeXAtom" data-mjx-texclass="ORD"><g data-mml-node="mpadded"><g data-mml-node="munder"><g data-mml-node="TeXAtom" data-mjx-texclass="OP"><g data-mml-node="munder"><g data-mml-node="TeXAtom" data-mjx-texclass="ORD"><g data-mml-node="mphantom"></g></g><g data-mml-node="mo" transform="translate(0, -588)"><use xlink:href="#MJX-2-TEX-S4-E152"></use><use xlink:href="#MJX-2-TEX-S4-E153" transform="translate(10162.7, 0)"></use><g data-c="E156" transform="translate(4856.4, 0)"><use xlink:href="#MJX-2-TEX-S4-E151"></use><use xlink:href="#MJX-2-TEX-S4-E150" transform="translate(450, 0)"></use></g><svg width="4606.4" height="720" x="350" y="-300" viewBox="1151.6 -300 4606.4 720"><use xlink:href="#MJX-2-TEX-S4-E154" transform="scale(17.274, 1)"></use></svg><svg width="4606.4" height="720" x="5656.4" y="-300" viewBox="1151.6 -300 4606.4 720"><use xlink:href="#MJX-2-TEX-S4-E154" transform="scale(17.274, 1)"></use></svg></g></g></g><g data-mml-node="mtext" transform="translate(4538.8, -1488) scale(0.707)"><use xlink:href="#MJX-2-TEX-N-73"></use><use xlink:href="#MJX-2-TEX-N-61" transform="translate(394, 0)"></use><use xlink:href="#MJX-2-TEX-N-6D" transform="translate(894, 0)"></use><use xlink:href="#MJX-2-TEX-N-65" transform="translate(1727, 0)"></use></g></g></g></g><g data-mml-node="msub"><g data-mml-node="mi"><use xlink:href="#MJX-2-TEX-I-1D460"></use></g><g data-mml-node="mn" transform="translate(469, -150) scale(0.707)"><use xlink:href="#MJX-2-TEX-N-30"></use></g></g><g data-mml-node="mo" transform="translate(1039.2, 0)"><use xlink:href="#MJX-2-TEX-N-2026"></use></g><g data-mml-node="msub" transform="translate(2377.9, 0)"><g data-mml-node="mi"><use xlink:href="#MJX-2-TEX-I-1D460"></use></g><g data-mml-node="TeXAtom" transform="translate(469, -150) scale(0.707)" data-mjx-texclass="ORD"><g data-mml-node="mi"><use xlink:href="#MJX-2-TEX-I-1D456"></use></g><g data-mml-node="mo" transform="translate(345, 0)"><use xlink:href="#MJX-2-TEX-N-2212"></use></g><g data-mml-node="mi" transform="translate(1123, 0)"><use xlink:href="#MJX-2-TEX-I-1D459"></use></g></g></g><g data-mml-node="mo" transform="translate(4068.4, 0)"><use xlink:href="#MJX-2-TEX-N-2026"></use></g><g data-mml-node="mover" transform="translate(5407, 0)"><g data-mml-node="TeXAtom" data-mjx-texclass="OP"><g data-mml-node="mover"><g data-mml-node="mrow"><g data-mml-node="msub"><g data-mml-node="mi"><use xlink:href="#MJX-2-TEX-I-1D460"></use></g><g data-mml-node="mi" transform="translate(469, -150) scale(0.707)"><use xlink:href="#MJX-2-TEX-I-1D459"></use></g></g><g data-mml-node="mo" transform="translate(896.4, 0)"><use xlink:href="#MJX-2-TEX-N-2026"></use></g><g data-mml-node="msub" transform="translate(2235.1, 0)"><g data-mml-node="mi"><use xlink:href="#MJX-2-TEX-I-1D460"></use></g><g data-mml-node="mi" transform="translate(469, -150) scale(0.707)"><use xlink:href="#MJX-2-TEX-I-1D456"></use></g></g><g data-mml-node="mo" transform="translate(3164.7, 0)"><use xlink:href="#MJX-2-TEX-N-2026"></use></g><g data-mml-node="msub" transform="translate(4503.3, 0)"><g data-mml-node="mi"><use xlink:href="#MJX-2-TEX-I-1D460"></use></g><g data-mml-node="TeXAtom" transform="translate(469, -150) scale(0.707)" data-mjx-texclass="ORD"><g data-mml-node="mi"><use xlink:href="#MJX-2-TEX-I-1D45F"></use></g><g data-mml-node="mo" transform="translate(451, 0)"><use xlink:href="#MJX-2-TEX-N-2212"></use></g><g data-mml-node="mi" transform="translate(1229, 0)"><use xlink:href="#MJX-2-TEX-I-1D459"></use></g></g></g><g data-mml-node="mo" transform="translate(6268.8, 0)"><use xlink:href="#MJX-2-TEX-N-2026"></use></g><g data-mml-node="msub" transform="translate(7607.4, 0)"><g data-mml-node="mi"><use xlink:href="#MJX-2-TEX-I-1D460"></use></g><g data-mml-node="mi" transform="translate(469, -150) scale(0.707)"><use xlink:href="#MJX-2-TEX-I-1D45F"></use></g></g></g><g data-mml-node="mo" transform="translate(0, 702)"><use xlink:href="#MJX-2-TEX-S4-E150"></use><use xlink:href="#MJX-2-TEX-S4-E151" transform="translate(7995.3, 0)"></use><g data-c="E155" transform="translate(3772.7, 0)"><use xlink:href="#MJX-2-TEX-S4-E153"></use><use xlink:href="#MJX-2-TEX-S4-E152" transform="translate(450, 0)"></use></g><svg width="3522.7" height="720" x="350" y="-300" viewBox="880.7 -300 3522.7 720"><use xlink:href="#MJX-2-TEX-S4-E154" transform="scale(13.21, 1)"></use></svg><svg width="3522.7" height="720" x="4572.7" y="-300" viewBox="880.7 -300 3522.7 720"><use xlink:href="#MJX-2-TEX-S4-E154" transform="scale(13.21, 1)"></use></svg></g></g></g><g data-mml-node="mtext" transform="translate(3455.1, 1322) scale(0.707)"><use xlink:href="#MJX-2-TEX-N-73"></use><use xlink:href="#MJX-2-TEX-N-61" transform="translate(394, 0)"></use><use xlink:href="#MJX-2-TEX-N-6D" transform="translate(894, 0)"></use><use xlink:href="#MJX-2-TEX-N-65" transform="translate(1727, 0)"></use></g></g><g data-mml-node="mo" transform="translate(14019, 0)"><use xlink:href="#MJX-2-TEX-N-2026"></use></g></g></g></svg>
{{< /raw >}}

由于$r$只增不减，Z 算法的时间复杂度为$O(n)$。

参考代码（`std::string`，字符串下标从`0`开始）：
```cpp
void ZAlgorithm(const std::string &str)
{
    int len = str.length();
    z[0] = 0;
    for (int i = 1, l = 0, r = 0; i < len; i++)
    {
        int k = z[i - l];
        if (i <= r && k < r - i + 1)
            z[i] = k;
        else
        {
            k = std::max(r - i + 1, 0);
            while (i + k < len && str[i + k] == str[k])
                k++;
            z[i] = k;
        }

        if (i + k - 1 > r)
        {
            l = i;
            r = i + k - 1;
        }
    }
}
```

### Z 算法的扩展

有些时候，我们需要求某个字符串与另一个字符串后缀的最长公共前缀，该问题的解决方法同上，只是匹配时不再进行自身比较，而是文本串（的后缀）和模式串进行比较。

但要注意三个细节：
+ 仍然要取`z[i - l]`（这意味着要先对文本串做 Z 算法）。因为上面用于加速算法的性质是针对一个字符串的，不能用新的代替。
+ 匹配时注意边界，文本串和模式串都要判断。
+ 单独处理$z_0$。单个字符串与自己的最大公共前缀一定是它的长度（Z 函数中只是定义为$0$）扩展处理时如果不定义$z_0 = 0$，应当从头枚举计算$z_0$。

参考代码（`std::string`，字符串下标从`0`开始）：
```cpp
// `prev` means the `z` of `text`.
void ExZAlgorithm(const std::string &text, const std::string &pattern)
{
    int tLen = text.length(), pLen = pattern.length();
    z[0] = 0;
    for (int i = 0; i < std::min(tLen, pLen); i++)
    {
        if (text[i] == pattern[i])
            z[0]++;
        else
            break;
    }

    for (int i = 1, l = 0, r = 0; i < tLen; i++)
    {
        int k = prev[i - l]; // not `z`.
        if (i <= r && k < r - i + 1)
            z[i] = k;
        else
        {
            k = std::max(r - i + 1, 0);
            while (i + k < tLen && k < pLen && text[i + k] == pattern[k])
                k++;
            z[i] = k;
        }

        if (i + k - 1 > r)
        {
            l = i;
            r = i + k - 1;
        }
    }
}
```
