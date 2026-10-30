# Laporan Bedah REA — cn.cardoor.d3dview (D3Dview)

Dibedah 2026-10-30 dengan REA (rea-agents 6.8.0, provider JADX headless 0.7.1):
`inspect-android-package` (evidence `ev_f26ffbbd64a361024cf1addbc7c9ff7bd251495bacba62f975e27205f8d6c86b`)
dan `inspect-apk-signature` (evidence `ev_066721dc6b71c5723eda1baefa68e1637d447c0fd2648e4e333d24ca66f1048`).

## Identitas

| Item | Nilai |
|---|---|
| Paket | cn.cardoor.d3dview |
| Nama tampil | 3DView (@string/app_3DView) |
| Versi | V3.1.15.70_260819 (code 70) |
| minSdk / targetSdk | 21 / 27 |
| Ukuran | 71.226.357 byte, 1.479 entri zip |
| SHA-256 | 2241d90914e01c6c3584b3519620a29ee8b3b2c0679bef44081ec8991d77f79f |
| Sumber | /data/app/.../base.apk di HU uis7862a_1h10_Natv (Android 12) |

## Tanda tangan

- Skema v1 (JAR) saja. File: META-INF/DFASSSIS.SF / DFASSSIS.RSA.
- Penandatangan: sertifikat self-signed `CN=Zheng, O=jk, OU=jkkj, L=gd, ST=sz, C=457`,
  serial 57C4FBE2, berlaku 2016-08-30 s.d. 2041-08-24, RSA.
- SHA-256 sertifikat: 5b88187716d74bca641b35da261cfa52f86467ddda0ac6099083184df842999d.

## Isi APK (yang membuat ukurannya 68 MB)

Mayoritas adalah aset mesin render 3D (total assets ±147 MB sebelum kompresi):

- Tekstur langit/cuaca KTX sangat besar: `assets/engine/weather/cloud.ktx` 48 MB,
  skybox HDR siang & malam masing-masing 18 MB, tekstur radiance ±12 MB ×2,
  `thunder0.png` 12,7 MB, `fog_1.png` 3,6 MB.
- Model mobil 3D: `assets/engine/download/carmodel/.../auto3d_carmodel.glb` 3,6 MB.
- Native: `lib/armeabi-v7a/libnative-lib.so` 5,9 MB (hanya ABI 32-bit armeabi-v7a).
- `classes.dex` 6,1 MB — lihat catatan packer di bawah.

## Komponen penting (dari AndroidManifest)

- **Auto3dHomeActivity** (`cn.cardoor.d3dview.Auto3dHomeActivity`) + alias LAUNCHER
  `cn.cardoor.auto3d.Auto3dHomeActivity` — pintu masuk tampilan mobil 3D.
- **GyroscopeActivity** — tampilan 3D mengikuti sensor giroskop.
- **BridgeService** (`cn.cardoor.vehicle3d.bridge.server.BridgeService`, exported) —
  jembatan data kendaraan 3D: intent `VEHICLE3D_BRIDGE_SERVER` dan
  `ATTITUDE3D_BRIDGE_SERVER`; inilah jalur yang dipakai desktop Cardoor untuk
  menampilkan mobil 3D melayang (floating window "phantom",
  `VarietyFloatingAppWindow`, layanan `cn.cardoor.floating.app.window.s`).
- **Toko bawaan** `com.cardoor.shop.*` — ShopActivity, OpenMembershipActivity,
  ModelDetail/SceneDetail/AccessoryDetail: jualan model mobil, scene, aksesori 3D.
- **Akun** `cn.cardoor.user.*` — Login/Agreement/Logout + AccountService.
- Lokasi Baidu (`com.baidu.location.f`), AgentWeb, GMS.
- Paket yang di-query: com.autonavi.amapauto, com.baidu.BaiduMap.auto, com.tw.dvr,
  cn.cardoor.zt360, com.dofun.recorder/recorder2k, com.dofun.dofunweather.main.

## Izin (20)

Standar: INTERNET, READ/WRITE_EXTERNAL_STORAGE, lokasi (fine/coarse/background +
extra commands), ACCESS_NETWORK_STATE, ACCESS/CHANGE_WIFI_STATE, READ_PHONE_STATE,
QUERY_ALL_PACKAGES, REQUEST_INSTALL_PACKAGES, FOREGROUND_SERVICE.
Khusus/sensitif: SYSTEM_ALERT_WINDOW (jendela melayang), READ_LOGS, DUMP,
STOP_APP_SWITCHES, dan dua izin kustom antar-aplikasi vendor:
`com.dofun.variety.permission.THEME_LINKAGE_SERVICE` (kaitan tema dengan launcher
Cardoor) dan `com.tw.dvr.permission.ACCESS_ADAS_SERVICE` (layanan ADAS DVR TW).

## Catatan packer (penting untuk analisis lanjutan)

Application class adalah `com.stub.StubApp` dan JADX hanya memuat **5 kelas** dari
classes.dex 6,1 MB — ciri khas APK **ter-packer** (kode asli terenkripsi dan dimuat
saat runtime oleh stub). Bedah statis kode Java/Kotlin yang sebenarnya memerlukan
unpacked dump dari perangkat saat aplikasi berjalan; analisis manifest, aset,
native, dan permukaan IPC di laporan ini tidak terpengaruh packer.

## Relevansi ke proyek launcher custom

- Mobil 3D di desktop Cardoor berasal dari aplikasi ini lewat BridgeService —
  bukan bagian dari launcher itu sendiri.
- Izin THEME_LINKAGE_SERVICE menunjukkan D3Dview ikut terkait tema launcher bawaan.
- Aset 3D-nya (model GLB, skybox, cuaca) adalah bagian terberat; launcher custom
  tidak membutuhkannya dan tidak terpengaruh ada/tidaknya aplikasi ini.
