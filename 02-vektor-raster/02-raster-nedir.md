# Raster nədir

## Nə

**Raster** yer üzünü **bərabər kvadratlara bölüb hər kvadrata bir ədəd yazmaq** üsuludur. Bu kvadratlara **piksel** deyilir.

Rasterdə obyekt anlayışı yoxdur. "Bu yoldur, bu binadır" yazılmır — sadəcə ədədlər şəbəkəsi var. Yolu rasterdə görürsən, çünki onun pikselləri boz çıxıb, amma fayl bilmir ki, orada yol var.

Bir raster faylı üç şeydən ibarətdir:

**1. Şəbəkənin ölçüsü** — neçə sətir, neçə sütun. Məsələn `4000 × 4000` = 16 milyon piksel.

**2. Geotransform** — 6 ədəddən ibarət qayda. Şəbəkəni yer üzünə bağlayan yeganə şey budur:

| Ədəd | Nədir |
|---|---|
| 1 | sol-üst pikselin sol-üst **küncünün** uzunluğu (X) |
| 2 | bir pikselin eni (məsələn 0.25 m) |
| 3 | fırlanma (adətən 0) |
| 4 | sol-üst pikselin sol-üst **küncünün** enliyi (Y) |
| 5 | fırlanma (adətən 0) |
| 6 | bir pikselin hündürlüyü (adətən mənfi, çünki yuxarıdan aşağı sayılır) |

Bu altı ədəd olmasa fayl sadəcə şəkildir — yerin hansı hissəsi olduğu bilinmir.

**3. Zolaqlar (band)** — şəbəkənin neçə qatı var. Hər qat eyni piksellər üçün fərqli ölçmədir:

| Zolaq sayı | Nümunə |
|---|---|
| 1 | hündürlük modeli (hər piksel = metr) |
| 3 | rəngli şəkil (qırmızı, yaşıl, mavi) |
| 4 | rəng + şəffaflıq |

**Piksel nömrəsindən koordinat necə çıxır?**

```
lon = origin_lon + (sütun + 0.5) × piksel_eni
lat = origin_lat − (sətir  + 0.5) × piksel_hündürlüyü
```

`+ 0.5` ona görədir ki, piksel nöqtə deyil, **kvadratdır**. İçindəki ədəd bütün kvadrata aiddir. Onu bir nöqtə ilə təmsil edəcəksənsə, ədalətli nöqtə **mərkəzdir**. Üstəlik bir künc 4 pikselin arasında bölüşülür — "bu koordinat hansı pikselindir?" sualı cavabsız qalır. Mərkəz isə təkdir.

## Niyə

1. **Bəzi şeyin sərhədi yoxdur.** Hündürlük, temperatur, nəmlik, torpaq növü — bunlar yer üzündə davamlı dəyişir, "obyekt" kimi kəsilmir. Onları nöqtə-xətt-poliqonla yazmaq olmur, hər nöqtə üçün dəyər lazımdır.
2. **Şəkil özü mənbədir.** Təyyarədən və peykdən çəkilən görüntü təbiətcə pikseldir. Vektor ondan **sonra** çıxarılır: insan ya proqram şəkilə baxıb yolu cızır.
3. **Oxumaq sadədir.** Sətir-sütun bilirsənsə, dəyəri birbaşa götürürsən. Axtarış lazım deyil.

## Sadə nümunə

Uydurma bir hündürlük rasteri, cəmi `3 × 3` piksel, piksel ölçüsü 10 metr. Sol-üst küncü `47.500 40.500`.

Dəyərlər (metrlə):

```
sütun →      0     1     2
sətir 0:   120   122   125
sətir 1:   118   121   124
sətir 2:   115   117   120
```

**Sual:** `sətir 1, sütun 2` pikselinin mərkəzi hansı koordinatdadır?

10 metr dərəcə ilə təxminən `0.0001`-dir (sadəlik üçün belə götürək):

```
lon = 47.500 + (2 + 0.5) × 0.0001 = 47.50025
lat = 40.500 − (1 + 0.5) × 0.0001 = 40.49985
dəyər = 124 metr
```

İndi vacib məqamı gör: bu şəbəkədə **"təpə" obyekti yoxdur**. 125 metr ən yüksək dəyərdir, amma faylda "təpə" adlı heç nə yazılmayıb. Təpəni görmək üçün kimsə şəbəkəyə baxıb qərar verməlidir.

## Tipik səhv

**1. Geotransform-u itirmək.**
`.tif` faylını adi şəkil redaktorunda açıb yadda saxlasan, proqram altı ədədi atır. Fayl açılır, görüntü yerindədir, amma yerə bağlı deyil. Belə fayl xəritədə ya heç görünmür, ya da Afrika sahillərində — çünki koordinatsız rasterin başlanğıcı `0 0` sayılır.

**2. Piksel mərkəzi ilə küncünü qarışdırmaq.**
`+ 0.5` yazmasan, hər koordinat yarım piksel sürüşür. 25 sm pikseldə bu 12.5 sm-dir — az görünür, amma vektor layı ilə üst-üstə qoyanda sistematik sürüşmə kimi çıxır.

**3. "Fayl kiçikdir, çünki ərazi boşdur" düşünmək.**
Raster həcmi məzmundan asılı deyil. Tamam ağ, tamam boş ərazi də bütün pikselləri tutur. Həcmi yalnız iki şey müəyyən edir: **ərazinin böyüklüyü** və **piksel ölçüsü**. Piksel ölçüsünü iki dəfə kiçiltsən, fayl **4 dəfə** böyüyür.

## Xülasə

Raster yer üzünü bərabər kvadratlara bölüb hər birinə bir ədəd yazır — obyekt anlayışı yoxdur, şəbəkəni yerə bağlayan yeganə şey geotransform-un altı ədədidir, həcmi isə məzmundan deyil, ərazi ilə piksel ölçüsündən asılıdır.
