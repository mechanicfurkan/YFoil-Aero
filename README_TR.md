# YFoil Aero

**Tarayıcı tabanlı kanat aerodinamiği simülasyon aracı.**  
Prandtl Kaldırma Hattı Teorisi, viskoz-inviskoz panel çözücü ve gerçek NACA profil veritabanı kullanarak kırık kanat geometrilerini analiz eder.

> **Amaç:** Eğitim ve ön tasarım. Sertifikasyon veya uçuş kritik mühendislik kararlarında tek başına kullanılmamalıdır.
![YFoil Aero Demo](Foilteest.gif)
---

## Ne Yapar?

Bir kanat geometrisi tanımlarsın (açıklık, kiriş, süpürme, burulma, NACA profili), uygulama anlık olarak şunları hesaplar:

- **CL, CD, CDi, Cm** — kaldırma, sürükleme, indüklü sürükleme, yunuslama momenti
- **L/D oranı** — aerodinamik verim
- **Kanat açıklığı boyunca yük dağılımı** — spanwise CL eğrisi
- **Basınç katsayısı dağılımı** — Cp grafiği
- **Durma açısı ve durma tipi** — ön kenar (LE) vs. arka kenar (TE) ayrımı
- **Statik marj ve nötr nokta** — boyuna denge analizi
- **Kök eğilme momenti** — yapısal ön boyutlandırma
- **Dalga sürüklemesi** — Mach > 0.5 rejimleri için
- **Yer etkisi** — AGL yüksekliği parametresi ile

XFOIL polar dosyası (.txt/.dat) import edilerek veri tabanındaki NACA profillerinin dışına çıkılabilir.

---

## Fizik Motoru

| Modül | Yöntem | Referans |
|---|---|---|
| Kaldırma hattı teorisi | Doğrusal olmayan LLT (30 Fourier modu) | Prandtl (1918), McCormick (1979) |
| Panel çözücü | Hess-Smith kaynak-girdap yöntemi + Head sınır tabakası | Hess & Smith (1967) |
| Profil veri tabanı | NACA deneysel verileri (Re interpolasyonu) | Abbott & von Doenhoff (1959) |
| Geçiş tahmini | Michel kriteri | Michel (1951) |
| Dalga sürüklemesi | Kármán-Tsien düzeltmesi + Mcr modeli | Drela, XFOIL dokümantasyonu |
| Atmosfer modeli | ISA standardı (0–20 km) | ICAO Doc 7488 |
| Flap etkisi | NACA ΔCL_δ yöntemi | Raymer (1992) |
| Durma modeli | LE/TE stall tipi ayrımı, cos²·exp geçiş sonrası CL | McCormick (1979) |

---

## Doğruluk ve Sınırlamalar

Bu araç ön tasarım ve eğitim için geliştirilmiştir. Sonuçlar **teorik hesaplamaya** dayanmaktadır.

**En güvenilir rejim:**
- En boy oranı (AR) > 4
- Süpürme açısı < 25°
- Mach < 0.6
- Hücum açısı < stall açısı

Bu koşullarda CL hatası tipik olarak ±%3–5, CDi hatası ±%5–10 düzeyindedir; bu değerler havacılık mühendisliği ön tasarım standardıyla uyumludur.

**Dikkat edilmesi gereken durumlar:**
- AR < 4 veya yüksek süpürme açısı (> 30°): LLT'nin teorik sınırları aşılır, hata büyür
- Stall sonrası bölge: eğilim doğrudur fakat sayısal değerler kesin değildir
- Re < 200.000 (model uçak, nano-UAV): veri tabanı bu rejimi kapsamaz; XFOIL polar import kullanılmalıdır

Ayrıntılı doğruluk analizi için: [`YFoil_Dogruluk_Raporu.docx`](./YFoil_Dogruluk_Raporu.docx)

---

## Kullanım

Herhangi bir kurulum gerekmez. Tek HTML dosyasıdır:

```
YFoil_Aero_v87.html dosyasını tarayıcıda aç.
```

Canlı dene: **[https://mechanicfurkan.github.io/YFoil-Aero/](#)**  
https://mechanicfurkan.github.io/YFoil-Aero/ 

---

## Versiyon Geçmişi

| Versiyon | Açıklama |
|---|---|
| v5.x | İlk çalışan prototip, DATCOM tabanlı kaldırma modeli |
| v6.0 | CD çift sayım hatası düzeltildi, post-stall cos²·exp eğrisi |
| v7.0 | LLT'ye geçiş, gerçek NACA PDB, bilineer Re+α interpolasyonu |
| v7.5 | VIE viskoz-inviskoz panel çözücü, 2D rüzgar tüneli görünümü |
| v7.9 | Panel sistemi sadeleştirildi, CDf çift sayım giderildi, al0 önbelleği |

---

## Lisans

**Creative Commons Atıf-GayriTicari 4.0 (CC BY-NC 4.0)**

Kullanabilirsin · Paylaşabilirsin · Uyarlayabilirsin  
Kaynak göstermek zorunludur · Ticari kullanım yasaktır

Tam metin: [LICENSE](./LICENSE)

---

## Yazar

Bu proje, genç havacılık mühendislerine sezgisel ve referans kalitesinde bir araç sunmak amacıyla Yıldız Teknik Üniversitesi Makine Mühendisliği öğrencisi tarafından geliştirilmiştir. Created by M. Furkan YILMAZ.

Referans literatür: Abbott & von Doenhoff (1959) · McCormick (1979) · Raymer (1992) · Drela XFOIL
