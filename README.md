# cumpc-mujoco

[CuMPC](../CuMPC) custom CUDA MPPI çekirdeğinin **MuJoCo geliştirme/simülasyon kabuğu**.
Differential-drive bir tarım robotu MuJoCo'da simüle edilir; tüm kontrol matematiği
CuMPC çekirdeğinde (GPU) koşar — bu repo yalnız sensör I/O, aktüatör dönüşümü,
referans path üretimi, metrik ve görselleştirme içerir.

Sonuçlar: [docs/RESULTS.md](docs/RESULTS.md) · Ortam raporu: [docs/PHASE0_REPORT.md](docs/PHASE0_REPORT.md)

## Sonuç özeti (RTX 4070, K=2048, H=40, 20 Hz kontrol)

| Metrik | Hedef | Ölçülen |
|---|---|---|
| Cross-track RMS (düz / eğri) | < 0.03 m | 0.007 / 0.025 m |
| Cross-track RMS (S-curve) | < 0.05 m | 0.028 m |
| Collision (engel + terrain) | 0 | 0 |
| MPPI loop-rate | ≥ 20 Hz | ~4000 Hz (step ≈ 150 µs) |
| Slip-aware vs naive (yamaç) | daha düşük RMS | 0.075 vs 0.107 m (−%30) |

## Gereksinimler

- NVIDIA GPU + CUDA Toolkit ≥ 12.x (`nvcc` PATH'te) — çekirdek kurulumda derlenir
- Python ≥ 3.10

## Kurulum

### Yol 1 — Sadece çalıştırmak için (pip + git)

CuMPC çekirdeği doğrudan GitHub'dan kurulur:

```bash
git clone https://github.com/ahmet-f-gumustas/cumpc-mujoco.git
cd cumpc-mujoco
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
pip install "cumpc @ git+https://github.com/ahmet-f-gumustas/CuMPC.git@v0.1.0"
python examples/track_scurve.py --viewer
```

> Çekirdek varsayılan olarak `sm_89` (RTX 40xx) için derlenir. Farklı GPU'da:
> `CMAKE_ARGS="-DCMAKE_CUDA_ARCHITECTURES=86" pip install "cumpc @ git+..."`
> (RTX 30xx için 86; `nvidia-smi --query-gpu=compute_cap --format=csv` ile öğrenilir).
>
> Not: `v0.1.0` etiketi CuMPC çekirdeği yayınlandığında aktif olur.

### Yol 2 — Çekirdeği de geliştirmek için (yan yana klon + editable)

```bash
git clone https://github.com/ahmet-f-gumustas/CuMPC.git
git clone https://github.com/ahmet-f-gumustas/cumpc-mujoco.git
cd cumpc-mujoco
uv venv --python 3.10 && source .venv/bin/activate
uv pip install -r requirements.txt
uv pip install -e ../CuMPC        # çekirdek değişince aynı komutla yeniden derlenir
```

## Çalıştırma

```bash
python examples/drive_manual.py --viewer        # manuel sürüş (kontrolcüsüz)
python examples/track_straight.py --viewer      # düz path takibi
python examples/track_curve.py --viewer         # eğri
python examples/track_scurve.py --viewer        # S-curve
python examples/full_course.py --no-terrain     # engelli kurs (düz zemin)
python examples/full_course.py --viewer         # engel + terrain tam kurs
python examples/slope_traverse.py               # slip-aware vs naive karşılaştırma
```

Headless koşu için `MUJOCO_GL=egl` ekleyin (`--viewer` olmadan).

## Metrik / grafik

```bash
python metrics/evaluate.py --all          # runs/*.npz → metrik tablosu
python metrics/plot_results.py --all      # docs/plots/*.png
```

## Yapı

```
mujoco_harness/   sahne XML + sensör/aktüatör/path/koşu döngüsü (yalnız dış kabuk)
config/           default.yaml — MPPI + robot + cost + harita + sim parametreleri
examples/         koşu senaryoları
metrics/          değerlendirme + grafik
docs/             sonuçlar, ortam raporu, grafikler
```

## Lisans

GNU General Public License v3.0 — bkz. CuMPC reposundaki [LICENSE](../CuMPC/LICENSE).
