# **YFoil Aero v7.9**

## **Tarayıcı Tabanlı Kanat Aerodinamiği Simülatörü**

### **Teknik Tasarım, Fizik Motoru ve Doğruluk Raporu**

**Hazırlayan:**  
Furkan Yılmaz  
Makine Mühendisliği  
2025

## **Özet**

YFoil Aero, havacılık ve makine mühendisliği alanında kanat aerodinamiğini analiz etmeyi amaçlayan, tamamen tarayıcı içinde çalışan, tek dosyadan oluşan bir simülasyon aracıdır. Geliştirme sürecindeki temel motivasyon; kurulum gerektiren ve ağır hesaplama altyapılarına ihtiyaç duyan klasik aerodinamik analiz yazılımlarına (XFOIL, OpenVSP, DATCOM gibi) hafif, erişilebilir ve eğitim odaklı bir alternatif sunmaktır.  
Bu rapor, uygulamanın fizik motorunda kullanılan matematiksel ve deneysel modelleri, bu modellerin teorik temellerini, uygulamanın hassasiyetlerini ve deneysel verilere karşı karşılaştırmalı bir doğruluk analizini kapsamlı bir şekilde ele almaktadır. Rapor aynı zamanda bir tasarım ve geliştirme dokümanı işlevi görerek; hangi modülün neden seçildiğini, sınırlarının nerede başladığını ve hangi koşullar altında güvenilir sonuçlar ürettiğini açıklar.  
Uygulamanın çekirdeğini oluşturan fizik modülleri şunlardır: Uluslararası Standart Atmosfer (ISA) modeli, Hess-Smith kaynak-girdap panel yöntemi ve Head integral sınır tabaka çözücüsünden oluşan viskoz-invizcid etkileşimi (VIE) altyapısı, sağlam bir 3-Katmanlı Aerodinamik Çözücü (Vorteks Kafes Yöntemi, Lineer Olmayan Taşıma Çizgisi Teorisi ve Lineer Taşıma Çizgisi Teorisi), Abbott ve von Doenhoff'un (1959) deneysel NACA polar veritabanı, Karman-Tsien ve Laitone harmanlanmış sıkıştırılabilirlik düzeltmeleri, dalga sürüklemesi modeli, Michel geçiş kriteri ve Schlichting hibrit sınır tabaka sürtünme formülasyonu.  
Doğruluk değerlendirmesi sonucunda; hesaplanan taşıma katsayısı (CL), indüklenmiş sürükleme katsayısı (CDi) ve açıklık boyunca yük dağılımının, lineer aerodinamik rejimde (açıklık oranı AR > 4, ok açısı < 25°, Mach < 0.6, hücum açısı < stall açısı) endüstri standardı araçlarla karşılaştırılabilir doğruluğa sahip sonuçlar ürettiği gözlemlenmiştir. Yüksek ok açısı, düşük açıklık oranı ve stall sonrası rejim, kullanılan fiziksel modellerin doğal sınır koşullarını oluşturur.

| Parametre | Değer |
| :---- | :---- |
| Uygulama Sürümü | YFoil Aero v7.9 |
| Platform | Tarayıcı (Vanilla JS, HTML5, Plotly.js) |
| 2D Panel Çözücü | Hess-Smith Kaynak-Girdap (80 panel) |
| 3D Çözücü Mimarisi | 3-Katmanlı Fallback (VLM → NLLT → pLLT) |
| VLM Ayrıklaştırması | İnce Kanat At Nalı Girdapları (20 açıklık paneli) |
| NLLT Ayrıklaştırması | 30 Fourier Modu |
| Sınır Tabaka | Head İntegral Yöntemi (VIE) |
| Polar Veritabanı | Abbott & von Doenhoff (1959) — 18 NACA Profili |
| Sıkıştırılabilirlik | Harmanlanmış Karman-Tsien ve Laitone Düzeltmeleri |
| Dinamik Kararlılık | Analitik $C_{L_q}$ ve $C_{m_q}$ takibi |
| Re Aralığı (PDB) | 500,000 – 6,000,000 |
| Mach Geçerlilik Sınırı| M < 0.85 (dalga sürüklemesi modeli) |
| Güvenilir Rejim | AR > 4, Λ < 25°, M < 0.6, α < α_stall |

## **1\. Giriş ve Motivasyon**

### **1.1 Problem Tanımı**

Kanat aerodinamiği analizi, havacılık mühendisliğindeki en temel ve zorlu problemlerden birini oluşturur. Gerçek akış koşullarının doğru modellenmesi; türbülanslı sınır tabakalar, akış ayrılması, sıkıştırılabilirlik etkileri ve üç boyutlu girdap yapıları gibi karmaşık fiziksel fenomenlerin aynı anda ele alınmasını gerektirir. Bu nedenle, endüstriyel düzeyde doğruluk çoğunlukla pahalı rüzgar tüneli deneyleri veya hesaplamalı akışkanlar dinamiği (CFD) aracılığıyla elde edilir.  
Öte yandan, mühendislik eğitiminde ve erken tasarım aşamalarında, fiziksel sezgi geliştirmek ve hızlı parametrik iterasyon sağlamak, tam bir CFD hesaplamasının sağladığı mutlak hassasiyetten daha önceliklidir. Bu ihtiyacı karşılayan mevcut araçların büyük çoğunluğu ya ağır kurulum prosedürleri gerektirir, işletim sistemine bağımlı kalır ya da programlama bilgisi olmadan kullanılamaz. YFoil Aero bu boşluğu doldurmak için geliştirilmiştir.

### **1.2 Tasarım Hedefleri**

Projenin tasarım sürecinde belirlenen hedefler üç ana başlık altında özetlenebilir:

* **Erişilebilirlik:** Herhangi bir kurulum veya arka uç (backend) altyapısı gerektirmeden yalnızca bir web tarayıcısı kullanılarak çalışabilmek. Tüm hesaplamalar istemci tarafında (tarayıcıda) gerçekleşir.  
* **Fiziksel Doğruluk:** Kullanılan modeller yalnızca ampirik regresyonlardan oluşmamalıdır; bunun yerine fizik tabanlı yarı analitik yöntemler (VLM, panel çözücü, integral sınır tabaka), deneysel verilerle (Abbott & von Doenhoff PDB) birleştirilmiştir.  
* **Şeffaflık:** Doğruluk sınırlarının ve her modelin hangi koşullar altında geçerli olduğunun açıkça belgelenmesi. Bu raporun temel amacı budur.

### **1.3 Kapsam ve Sınırlamalar**

YFoil Aero aşağıdaki analiz yeteneklerini sunar:

* Kırık kanat (cranked) geometrisi: kink (kırılma) konumu, iç/dış hücum kenarı ok açısı, dihedral, burulma (twist), sivrilme (taper)  
* Anlık CL, CD, CDi, Cm hesaplaması ve taşıma/sürükleme kuvvetleri  
* α = -10° ile +20° arasında tam polar eğri taraması  
* Açıklık boyunca (spanwise) yük dağılımı ve stall marjı  
* Basınç katsayısı (Cp) dağılımı — Hess-Smith panel çözücü veya ince kanat teorisi  
* Statik marjin, nötr nokta ve dinamik kararlılık türevleri (CLq, Cmq)  
* Kök eğilme momenti (root bending moment)  
* Dalga sürüklemesi ve Harmanlanmış Sıkıştırılabilirlik (M > Mcr)  
* Yer etkisi (Ground effect)  
* Flap etkisi (ΔCL, ΔCm, ΔCD)  
* XFOIL polar dosyası içe aktarımı  
* Mühendislik raporu çıktısı, CSV ve STL dışa aktarımı

Uygulamanın kapsamı dışında kalan konular şunlardır: Tam Navier-Stokes çözümü, türbülans modellemesi, dinamik stall, aeroelastik etkileşim (flutter, divergence), çoklu gövde (multi-body) konfigürasyonları ve itki (propulsive) etkileri.

## **2\. Atmosfer Modeli — Uluslararası Standart Atmosfer (ISA)**

### **2.1 Teorik Temel**

Atmosferik koşullar hava yoğunluğunu (ρ), dinamik viskoziteyi (μ) ve ses hızını (a) doğrudan etkilediğinden; Reynolds sayısı, Mach sayısı ve taşıma/sürükleme kuvvetlerinin hesaplanması için temel girdilerdir. YFoil Aero, ICAO Doc 7488 standardına uygun olarak Uluslararası Standart Atmosfer (ISA) modelini tam olarak uygular.

### **2.2 Uygulanan Denklemler**

**Troposfer (h ≤ 11.000 m):**  
T = 288.15 - 0.0065 · h [K]  
p = 101.325 · (T / 288.15)^5.2561 [kPa]  
**Alt Stratosfer (11.000 < h ≤ 20.000 m):**  
T = 216.65 [K] (izotermal)  
p = 22.632 · exp(-1.5769 × 10⁻⁴ · (h - 11000)) [kPa]

### **2.3 Doğruluk**

**ISA Modeli — Doğruluk Değerlendirmesi: Mükemmel**  
ICAO Doc 7488 referans tablolarıyla tam uyumludur. Sutherland yasası viskoziteyi ±%0.5 hassasiyetle hesaplar. Limit: Standart dışı atmosfer koşulları (tropikal, arktik) modellenmemiştir.

## **3\. NACA 4-Haneli Profil Koordinat Üreticisi**

### **3.1 Teorik Temel**

NACA 4 haneli profil serisi 1933 yılında Jacobs ve ark. tarafından tanımlanmıştır. Dört haneli kodun her biri şu bilgileri taşır: İlk hane kirişin yüzdesi olarak maksimum kamburluk değerini, ikinci hane kirişin onda biri cinsinden maksimum kamburluk konumunu, son iki hane ise kirişin yüzdesi olarak maksimum kalınlığı ifade eder.

### **3.2 Uygulanan Formülasyon**

Standart NACA 4 haneli kalınlık polinomu kullanılır. Panel çözücünün hassasiyetini artırmak için düğüm noktaları kosinüs dağılımı kullanılarak yoğunlaştırılır. Bu yöntemde, β = [0, π] arasında eşit örnekleme yapılarak x = 0.5(1 - cos β) dönüşümü uygulanır.

### **3.3 Doğruluk**

**NACA Koordinat Üreticisi — Doğruluk Değerlendirmesi: Mükemmel**  
Katsayılar orijinal NACA teknik raporlarından alınmıştır. Kamburluk-kalınlık eşleşmesi doğru bir şekilde uygulanmıştır. Limit: %21'in üzerindeki kalınlık değerleri için firar kenarı kapanma varsayımı bozulur. Limit: NACA 5 ve 6 serisi profiller üretilemez.

## **4\. Hess-Smith Kaynak-Girdap Panel Çözücüsü**

### **4.1 Tarihsel Arka Plan ve Motivasyon**

Panel yöntemleri 1960'ların sonlarında Smith ve Hess tarafından geliştirilmiş ve hesaplamalı aerodinamiğin temel araçlarından biri haline gelmiştir. YFoil Aero'da uygulanan Hess-Smith yöntemi (1967), profil sınırını N panele böler ve her panele sabit şiddette bir kaynak (source) ve üniform bir girdap (vortex) atar. Fiziksel olarak anlamlı taşıma üretimini sağlamak için firar kenarında Kutta koşulu uygulanır.

### **4.2 Matematiksel Formülasyon**

Her i-inci panel için kontrol noktasındaki normal hız bileşeni, doğrusal bir denklem sistemi ile ifade edilir. Her panel çiftinin etkisi, logaritmik ve trigonometrik terimleri birleştiren kapalı formül (closed-form) analitik ifadelerle hesaplanır. Kutta koşulu, üst ve alt firar kenarı panellerinin teğetsel hız bileşenlerinin toplamının sıfır olmasını sağlar.

### **4.3 Viskoz-İnvizcid Etkileşimi (VIE) Altyapısı**

Sadece invizcid (viskoz olmayan) bir panel çözücü sürükleme üretmez ve ayrılmayı modelleyemez. Bu sınırlamanın üstesinden gelmek için, Head integral sınır tabaka modeli panel çözücüyle gevşek bağlı (loosely coupled) bir yapıda kullanılır.

### **4.4 Head İntegral Sınır Tabaka Modeli**

Laminar-türbülans geçiş konumu, Michel (1951) kriterine benzer bir eşik koşulu ile belirlenir. Geçişten sonra Head'in sürüklenme (entrainment) yaklaşımı kullanılır. Viskoz sürükleme katkısı, firar kenarı momentum kalınlığı kullanılarak hesaplanır.

### **4.5 Doğruluk ve Sınırlamalar**

**Hess-Smith Paneli (İnvizcid) — Doğruluk: Çok İyi**  
80 panel için, Cp dağılımı referans panel çözücüleriyle (XFOIL) %1 içinde uyuşur. Limit: Tamamen invizcid yapıdadır — sürükleme (drag) üretmez.  
**Head Sınır Tabakası (VIE) — Doğruluk: Orta**  
Tutunmuş türbülanslı akışta (H < 2.0), θ ve H tahminleri niteliksel olarak doğrudur. Limit: Ters basınç gradyanı altında θ büyümesini tahmin etmek bu yöntemin zayıf noktasıdır.

## **5\. NACA Polar Veritabanı (PDB)**

### **5.1 Tarihsel Kaynak**

YFoil Aero'nun polar veritabanı, Abbott ve von Doenhoff'un 1959 tarihli 'Theory of Wing Sections' adlı referans kitabında yayınlanan Langley değişken yoğunluklu rüzgar tüneli verilerinden derlenmiştir.

### **5.2 İnterpolasyon Yöntemi**

α ekseni üzerinde doğrusal interpolasyon uygulanırken, Reynolds sayısı eksenindeki interpolasyon Re'nin logaritmik doğasını yansıtmak için log-uzayında gerçekleştirilir.

### **5.4 Doğruluk Analizi**

**NACA PDB — Doğruluk Değerlendirmesi: Çok İyi**  
Doğrudan birincil deneysel kaynaktan beslenir. Pre-stall ve orta-α değerleri için, CL hatası tipik olarak ±0.03-0.05'tir. Limit: Re < 200,000 için veri yoktur; ekstrapolasyon yapılır.

## **6\. Çekirdek Aerodinamik Motoru: 3-Katmanlı Fallback Çözücü**

3 boyutlu bir kanat üzerindeki taşıma ve indüklenmiş sürükleme dağılımını hesaplamak için YFoil Aero son derece sağlam 3 katmanlı bir fallback (yedekleme) mimarisi kullanır. Bu durum, simülatörün en gelişmiş 3D aerodinamik yönteme öncelik vermesini sağlarken, yakınsamanın başarısız olması durumunda daha basit modellere geri dönerek ekstrem uçuş rejimlerinde mutlak kararlılığı korumasını sağlar.

### **6.1 Katman 1: Vorteks Kafes Yöntemi (VLM)**
VLM, simülasyonun birincil motorudur ve kanat geometrisinin gerçek 3D modellemesini sunar.
* **Teorik Temel:** Açıklık boyunca (spanwise) yer alan panellere yerleştirilmiş at nalı (horseshoe) girdaplarını kullanan ince kanat teorisine dayanır.
* **Biot-Savart Yasası:** Her bir kontrol noktasındaki indüklenmiş hız, Biot-Savart yasası kullanılarak değerlendirilir. Yatay bağlı (bound) girdap segmentleri birincil taşıma kuvvetini üretirken, sonsuza uzanan iz (trailing) girdapları downwash (aşağı bastırma) etkisini modeller.
* **Çözüm Matrisi:** Panellere normal yönde sıfır akış (V·n = 0) sağlayan bir Sınır Koşulu etki matrisi oluşturulur. Bu matris, her panelin sirkülasyon şiddetini ($\Gamma$) belirlemek için Gauss Eleminasyonu ile çözülür.
* **Tekilliksiz İndüklenmiş Sürükleme (Trefftz Düzlemi):** Downwash'un doğrudan bağlı girdap kontrol noktalarında değerlendirilmesinden kaynaklanan devasa hata sıçramalarını (singularity) önlemek için YFoil Aero, Trefftz Düzlemi (Trefftz Plane) yaklaşımını uygular. İndüklenmiş sürükleme (CDi), yalnızca iz girdaplarından gelen downwash etkisi izole edilerek tam doğrulukla hesaplanır ve endüstriyel kalitede sürükleme tahminleri üretir.

### **6.2 Katman 2: Lineer Olmayan Taşıma Çizgisi Teorisi (NLLT)**
VLM çözücüsünün fizik dışı geometrilerle veya aşırı hücum açılarıyla karşılaşması durumunda motor NLLT'ye geri döner (fallback).
* **Fourier Serisi Formülasyonu:** Kanat boyunca sirkülasyon dağılımı $\Gamma(y)$, 30 modlu bir Fourier sinüs serisi olarak temsil edilir.
* **Lineer Olmayan İterasyon:** Yerel efektif hücum açısı hesaplanır ve stall başlangıcı ve viskoz etkiler gibi lineer olmayan olguları 3D hesaplamaya geri enjekte etmek için 2D profil veritabanı (PDB) sorgulanır. Döngü, $\Delta C_L < 10^{-4}$ toleransına ulaşılana kadar çalışır.

### **6.3 Katman 3: Lineer Taşıma Çizgisi Teorisi (pLLT)**
Eğer NLLT iterasyonu maksimum sınırına (18 iterasyon) yakınsamadan ulaşırsa, motor Prandtl'in klasik Lineer Taşıma Çizgisi Teorisine güvenir. Bu tek matrislik çözüm koşulsuz olarak kararlıdır ve nihai güvenlik ağı (fail-safe) görevi görür.

## **7\. Sürükleme Modelleri ve Sıkıştırılabilirlik**

Bir kanat üzerindeki toplam sürükleme dört temel bileşenden oluşur. YFoil Aero her birini ayrı modüller kullanarak hesaplar: CD = CD_profile + CD_friction + CDi + CD_wave.

### **7.1 Sıkıştırılabilirlik Düzeltmeleri**
Ses hızına ulaşılmadan önceki Mach etkilerini hesaba katmak için YFoil Aero, harmanlanmış (blended) bir sıkıştırılabilirlik düzeltmesi kullanır. Mach sayıları 0.55'e ve üstüne yaklaştıkça, çözücü **Karman-Tsien** kuralı ile **Laitone** düzeltme kuralı arasında kusursuz bir şekilde geçiş yapar. Bu harmanlanmış mantık, yüksek ses altı Mach sayılarında erken ıraksama (divergence) olmadan yerel basınç katsayılarının ($C_p$) ve taşıma eğimlerinin ($C_{L\alpha}$) doğru bir şekilde ölçeklenmesini sağlar.

## **8\. Stall Modeli**

2D CLmax değeri Re ile interpolasyon yoluyla PDB'den alınır. Stall sonrası CL ve CD modelleri, ani taşıma kaybını (Hücum Kenarı - LE stall) veya kademeli düşüşü (Firar Kenarı - TE stall) temsil etmek için birleştirilmiş ampirik ifadelerden yararlanır.

## **9\. Boylamasına Denge ve Dinamik Kararlılık Analizi**

YFoil Aero, hem statik hem de dinamik uçuş mekaniği parametreleri için anlık hesaplamalar gerçekleştirir.

### **9.1 Statik Marjin ve Nötr Nokta**
Boylamasına statik denge analizi, polar tarama verilerinden sayısal türev alınarak gerçekleştirilir. Nötr Nokta ($x_{np}$) hesaplanır ve Statik Marjin kullanıcı tanımlı Ağırlık Merkezine ($x_{cg}$) göre belirlenir. Kök eğilme momenti de açıklık boyunca yük dağılımı verileri kullanılarak yamuk (trapezoidal) integrasyonu yoluyla hesaplanır.

### **9.2 Dinamik Kararlılık Türevleri**
Uçuş simülasyonu ve kontrol sistemlerini desteklemek için aşağıdaki dinamik türevler analitik olarak çözülür:
* **Pitch-Rate Lift Türevi ($C_{L_q}$):** Uçağın yukarı doğru yunuslama (pitch) yapması nedeniyle oluşan ek taşıma kuvvetini temsil eder. Kanat boyunca indüklenen kamburluk (camber) etkisi modellenerek hesaplanır.
* **Pitch-Damping Türevi ($C_{m_q}$):** Boylamasına dinamik kararlılık için en hayati parametrelerden biridir ve yunuslama (pitch) osilasyonlarının ne kadar çabuk sönümleneceğini temsil eder. Kanat alanı, açıklık oranı (Aspect Ratio) ve aerodinamik merkez ile ağırlık merkezi arasındaki fark kullanılarak formüle edilmiştir.

## **10\. Hibrit VIE+PDB Polar Altyapısı**

PDB yalnızca sınırlı sayıda standart NACA profilini kapsar. Hibrit altyapı, PDB ve VIE sonuçlarını güvene dayalı ağırlıklı bir harmanlama (confidence-weighted blend) ile birleştirir. Bir XFOIL poları yüklendiğinde bu harmanlama tamamen devre dışı bırakılır ve doğrudan XFOIL verileri kullanılır.

## **11\. Genel Doğruluk ve Güvenilirlik Analizi**

Lineer aerodinamik rejimde (AR > 4, ok açısı < 25°, M < 0.6, α < stall), YFoil Aero, XFOIL ve OpenVSP gibi endüstri araçlarıyla karşılaştırılabilir doğrulukta sonuçlar üretir.

## **12\. Yazılım Mimarisi ve Teknik Uygulama**

### **12.1 Tek Dosyalı Uygulama Mimarisi**

YFoil Aero, kurulum gerektirmeyen ve taşınabilirliği en üst düzeye çıkaran tek dosyalı bir HTML yapısıyla geliştirilmiştir. Sıfır kurulum, platform bağımsızlığı ve statik barındırma (hosting) uyumluluğu ana avantajlarıdır.

### **12.2 Hesaplama Performansı**

Tek bir α değeri için tam hesaplama zinciri modern bir tarayıcıda yaklaşık 15-50 ms sürer. Bir polar taraması (61 nokta) yaklaşık 800-2.000 ms'de tamamlanır.

## **13\. Kullanım Kılavuzu**

index.html dosyasını herhangi bir modern tarayıcıda açın. Sol paneldeki kaydırıcıları (slider) kullanarak kanat geometrisini tanımlayın, uçuş koşullarını ayarlayın ve anlık sonuçları telemetri kartlarından okuyun.

## **14\. Sonuç ve Değerlendirme**

YFoil Aero v7.9, birden fazla fizik katmanını tek bir tarayıcı uygulamasında birleştiren kapsamlı bir kanat aerodinamiği simülatörüdür. Havacılık mühendisliği eğitimi ve ön tasarım aşaması için sağlam, şeffaf ve erişilebilir bir araç sunar.

## **Referanslar**

1. Abbott, I.H., von Doenhoff, A.E. (1959). Theory of Wing Sections. Dover Publications, New York.  
2. Prandtl, L. (1918). Tragflügeltheorie. Nachrichten von der Gesellschaft der Wissenschaften zu Göttingen.  
3. Hess, J.L., Smith, A.M.O. (1967). Calculation of potential flow about arbitrary bodies. Progress in Aerospace Sciences, 8, 1-138.  
4. Head, M.R. (1958). Entrainment in the turbulent boundary layer. ARC R\&M 3152, Her Majesty's Stationery Office.  
5. Michel, R. (1951). Etude de la transition sur les profils d'aile. ONERA Report 1/1578A.  
6. McCormick, B.W. (1979). Aerodynamics, Aeronautics and Flight Mechanics. Wiley, New York.  
7. Raymer, D.P. (1992). Aircraft Design: A Conceptual Approach, 2nd ed. AIAA Education Series.  
8. Schlichting, H. (1979). Boundary Layer Theory, 7th ed. McGraw-Hill, New York.  
9. Drela, M. (1989). XFOIL: An Analysis and Design System for Low Reynolds Number Airfoils. Low Reynolds Number Aerodynamics, Springer.  
10. Phillips, W.F. (2004). Lifting-Line Analysis for Twisted Wings and Washout-Optimized Wings. Journal of Aircraft, 41(1), 128-136.  
11. ICAO. (1993). Manual of the ICAO Standard Atmosphere, Doc 7488, 3rd ed.  
12. Reid, E.G. (1932). A Full-Scale Investigation of Ground Effect. NACA TN-265.  
13. Hoerner, S.F. (1965). Fluid-Dynamic Drag. Published by the author.  
14. Torenbeek, E. (1982). Synthesis of Subsonic Airplane Design. Delft University Press.  
15. Gallay, S., Laurendeau, E. (2015). Nonlinear Generalized Lifting-Line Coupling Algorithms for Pre/Poststall Flows. AIAA Journal, 53(7), 1784-1796.
