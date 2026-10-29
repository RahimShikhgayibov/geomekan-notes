# Koordinat və EPSG kodları

## Nə

**Koordinat** yer üzündə bir nöqtənin yerini göstərən ədədlərdir. Məsələn: `40.5, 47.5`.

Amma bu iki ədəd tək başına heç nə demir. Onları oxumaq üçün qayda lazımdır. Bu qaydaya **CRS** (Coordinate Reference System, koordinat istinad sistemi) deyilir. CRS bunları müəyyən edir:

- Yerin forması necə götürülüb (ellipsoid və datum, məsələn WGS 84)
- Vahid nədir: dərəcə, yoxsa metr
- Ədədlərin sırası necədir: əvvəl enlik, yoxsa əvvəl uzunluq
- Yer kürəsi düz müstəviyə necə köçürülüb (proyeksiya, əgər varsa)

CRS-lər iki böyük qrupa bölünür:

| Növ | Vahid | Nümunə |
|---|---|---|
| Coğrafi CRS | dərəcə (enlik, uzunluq) | EPSG:4326 (WGS 84) |
| Proyeksiyalı CRS | metr (x, y) | EPSG:3857 (Web Mercator), UTM zonaları |

**EPSG kodu** hər CRS-ə verilmiş nömrədir. Bu nömrələr EPSG reyestrində saxlanılır (indi onu IOGP aparır). "Bu data WGS 84 enlik/uzunluqdadır, ox sırası belədir..." deyə uzun izah yazmaq əvəzinə sadəcə `EPSG:4326` yazırsan.

Ən çox rast gələcəyin kodlar:

- **4326**: WGS 84, dərəcə. GPS və GeoJSON bunu işlədir.
- **3857**: Web Mercator, metr. Veb xəritələr (tile-lar) bunu işlədir.
- **326xx**: UTM şimal zonaları, metr. `xx` zona nömrəsidir, məsələn `32635` = UTM 35N.

## Niyə

1. **Eyni ədədlər fərqli CRS-də fərqli yer deməkdir.** CRS bilinmirsə, koordinat dəqiq yer göstərmir, sadəcə ədəd olaraq qalır.
2. **Laylar üst-üstə düşməlidir.** İki layın CRS-i fərqlidirsə, xəritədə biri o birinin yanında yox, tamam başqa yerdə görünür.
3. **Ölçü vahidi nəticəni dəyişir.** Məsafə və sahə metrlə hesablanmalıdır. Dərəcə ilə hesablanan "məsafə" kilometr deyil. Üstəlik 1 dərəcə uzunluq ekvatorda ~111 km, 40° enlikdə isə ~85 km edir.

## Sadə nümunə

Uydurma bir nöqtə götürək: **"Mərkəzi park"**. Eyni nöqtə iki fərqli CRS-də belə yazılır:

| CRS | Koordinat | Vahid |
|---|---|---|
| EPSG:4326 | uzunluq `47.5`, enlik `40.5` | dərəcə |
| EPSG:3857 | x `5 287 676`, y `4 938 869` | metr |

Yer eynidir, ədədlər tamam fərqlidir.

İndi təsəvvür et: **yollar layı** 4326-dadır, **binalar layı** isə 3857-də. İkisini CRS-i yoxlamadan eyni xəritəyə atsan, binalar milyonlarla "vahid" uzağa düşür. Proqram binaların ədədlərini dərəcə kimi oxuyur, nəticədə onlar ya xəritədən kənara çıxır, ya da tamam başqa yerdə görünür.

Düzgün yol: birini o birinin CRS-inə **çevirmək** (transform). PostGIS-də belə:

```sql
-- binalar layını 4326-ya çevir ki, yollar layı ilə üst-üstə düşsün
SELECT ST_Transform(geom, 4326) AS geom
FROM binalar;
```

## Tipik səhv

**1. Ox sırasını qarışdırmaq (enlik/uzunluq ↔ uzunluq/enlik).**
EPSG:4326-nın rəsmi tərifində sıra *enlik, uzunluq*-dur. Amma GeoJSON, WKT və PostGIS *uzunluq, enlik* (yəni x, y) yazır. Sıranı səhv salsan, `40.5, 47.5` nöqtəsi başqa yerə düşür. Bəzən dənizə, bəzən başqa ölkəyə.

**2. "CRS-i təyin etmək" ilə "CRS-ə çevirmək"i qarışdırmaq.**
- `ST_SetSRID(geom, 4326)` ədədlərə toxunmur, sadəcə üstünə "bu 4326-dır" etiketi yapışdırır. Yalnız CRS-i bilinməyən datanı etiketləmək üçündür.
- `ST_Transform(geom, 4326)` ədədlərin özünü yeni sistemə hesablayır.

Metrlə olan datanı `ST_SetSRID(..., 4326)` ilə "çevirməyə" çalışsan, `5 287 676` dərəcə uzunluq alınır. Belə bir yer yoxdur.

**3. 4326-da məsafə ölçmək.**
4326-da `ST_Distance` nəticəni metrlə yox, **dərəcə** ilə qaytarır. Metr lazımdırsa, ya metrik CRS-ə çevir (məsələn uyğun UTM zonası), ya da `geography` tipini işlət.

## Xülasə

Koordinat CRS-siz sadəcə ədəddir: EPSG kodu bu ədədlərin necə oxunacağını deyir, ona görə laylarla işləməzdən əvvəl hər birinin CRS-ini yoxla və lazım olanda `ST_SetSRID` ilə yox, `ST_Transform` ilə çevir.
