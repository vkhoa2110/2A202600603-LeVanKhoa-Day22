# Reflection - Lab 22 (DPO/ORPO Alignment)

**Ten:** Le Van Khoa  
**MSSV:** 2A202600603  
**Cohort:** TODO: dien cohort cua ban  
**Tier da chay:** T4 tren Colab - TODO cap nhat sau khi run-all thanh cong  
**Date:** 2026-06-26  
**Trang thai:** Ban nhap bao cao. Chua du dieu kien nop neu chua co artifact tu Colab.

---

## 1. Setup

| Item | Value |
|---|---|
| GPU local | NVIDIA GeForce RTX 2050 4GB - khong du de chay DPO 3B |
| GPU dung cho lab | TODO sau khi chay Colab: Free Colab T4 16GB / GPU thuc te |
| CUDA / driver | Local: NVIDIA driver 566.07, CUDA runtime hien thi 12.7 |
| Base model | `unsloth/Qwen2.5-3B-bnb-4bit` cho T4 tier |
| SFT dataset slice | `5CD-AI/Vietnamese-alpaca-cleaned`, 1000 samples, 1 epoch |
| Preference dataset slice | `argilla/ultrafeedback-binarized-preferences-cleaned`, TODO so pairs thuc te |
| `COMPUTE_TIER` env | `T4` |
| Total cost | TODO: `$0` neu dung free Colab T4 |

Ghi chu: minh khong chay local vi GPU RTX 2050 chi co 4GB VRAM. Theo hardware guide cua lab, DPO tier T4 can khoang 10-12GB VRAM vi phai tinh ca chosen/rejected va reference forward pass. Duong chay dung la Colab T4.

---

## 2. DPO experiment results

| Metric | SFT-only baseline | SFT + DPO |
|---|---:|---:|
| Training time (NB3) | - | TODO sau khi chay NB3 |
| VRAM peak | TODO | TODO |
| Final loss | TODO SFT loss cuoi NB1 | TODO DPO loss cuoi NB3 |
| Reward gap (chosen - rejected, end of training) | n/a | TODO lay tu `adapters/dpo/dpo_metrics.json` |
| Mean output length | TODO lay tu NB4 | TODO lay tu NB4 |

Tulu 3 reference numbers from deck are only context, not targets for this run: +1.7 MATH, +3.3 GSM8K, +1.3 IFEval at much larger scale. This lab uses a small 3B T4-friendly setting, so the important evidence is the before/after trend from my own SFT and DPO run.

---

## 3. Reward curves analysis

TODO: paste or link `submission/screenshots/03-dpo-reward-curves.png` after running Colab.

Ban phan tich nay se duoc cap nhat bang so lieu thuc sau khi NB3 chay xong. Dieu can doc khong chi la reward gap co tang hay khong, ma la tung duong `chosen_rewards` va `rejected_rewards` di theo huong nao. Neu `chosen_rewards` tang dan trong khi `rejected_rewards` giam hoac giu nguyen, DPO dang hoc dung tin hieu preference: cau tra loi duoc chon tro nen co log-ratio tot hon cau bi reject. Neu reward gap tang chu yeu vi `rejected_rewards` roi nhanh, con `chosen_rewards` cung giam, do co the la likelihood displacement nhu deck canh bao: model hoc day cau xau xuong manh hon la that su nang cau tot len. Khi do ket qua NB4 rat quan trong de xem output co ngan, an toan, va huu ich hon hay chi bi conservative.

Sau khi co plot, minh se dien ro: reward gap bat dau o muc nao, ket thuc o muc nao, chosen curve co tang hay giam, rejected curve co giam nhanh khong, va khoang nao cua training on dinh nhat. Neu nhung duong cong phang trong 100 step dau roi moi tach ra, day la hanh vi kha binh thuong voi DPO tren slice nho. Ket luan cuoi cung phai dua tren ca curve lan so sanh qualitative o NB4, vi reward gap duong khong tu dong co nghia la model tot hon voi prompt tieng Viet.

---

## 4. Qualitative comparison

TODO: paste or link `submission/screenshots/04-side-by-side-table.png` after running Colab.

| # | Prompt category | Prompt (truncated) | SFT-only | SFT+DPO | Winner |
|---|---|---|---|---|---|
| 1 | helpfulness | TODO | TODO | TODO | TODO |
| 2 | helpfulness | TODO | TODO | TODO | TODO |
| 3 | helpfulness | TODO | TODO | TODO | TODO |
| 4 | helpfulness | TODO | TODO | TODO | TODO |
| 5 | safety | TODO | TODO | TODO | TODO |
| 6 | safety | TODO | TODO | TODO | TODO |
| 7 | safety | TODO | TODO | TODO | TODO |
| 8 | safety | TODO | TODO | TODO | TODO |

**Win/loss/tie summary:** TODO sau khi NB4 tao `data/eval/judge_results.json`.  
**Judge used:** TODO: manual rubric / gpt-4o-mini / claude-haiku.

Tieu chi doc bang so sanh: voi helpfulness prompts, DPO nen tra loi co cau truc hon, it lan man hon, va gan voi yeu cau hon SFT-only. Voi safety prompts, DPO nen giu boundary ro hon, tranh dua huong dan nguy hiem truc tiep, nhung van nen dua loi khuyen thay the huu ich. Neu DPO qua ngan hoac tu choi qua muc, do la dau hieu alignment tax ve helpfulness.

---

## 5. Beta trade-off

| Beta | Reward gap | Win-rate (8 prompts) | Output length | Notes |
|---:|---:|---:|---:|---|
| 0.05 | TODO neu chay beta sweep | TODO | TODO | Conservative update |
| 0.1 default | TODO tu run chinh | TODO | TODO | Default cua lab |
| 0.5 | TODO neu chay beta sweep | TODO | TODO | Stronger KL constraint |

Minh chua chay beta sweep o ban nhap nay. Gia thuyet cua minh la beta 0.1 se la diem can bang tot nhat cho T4 tier: du manh de reward gap tach ra trong mot epoch, nhung khong qua aggressive de lam output tro nen ky hoac qua ngan. Beta 0.05 co the tao thay doi lon hon vi penalty voi reference nhe hon, nhung rui ro la model drift nhieu hon va de gap hien tuong length/style hacking. Beta 0.5 co the giu model gan SFT hon, vi vay output co the on dinh hon nhung reward gap va win-rate co kha nang tang it hon.

---

## 6. Personal reflection - single change that mattered most

Quyet dinh quan trong nhat cua minh trong lab nay la khong co gang chay pipeline tren laptop local, ma chuyen sang Colab T4. Lua chon thay the ban dau la chay truc tiep trong repo local vi moi thu da nam san trong may, nhung khi kiem tra `nvidia-smi`, GPU chi la NVIDIA GeForce RTX 2050 voi 4GB VRAM. Con so nay thap hon rat nhieu so voi yeu cau cua lab: T4 tier can khoang 10-12GB VRAM cho Qwen2.5-3B 4-bit DPO. Neu van co chay local, kha nang cao la minh se mat thoi gian cai dependency nang, tai model, roi gap OOM ngay o buoc load model hoac DPO training.

Minh chon Colab T4 vi do la duong chay duoc repo thiet ke san: notebook `Lab22_DPO_T4.ipynb` da set `COMPUTE_TIER=T4`, dung model 3B, batch size nho, va tao dung layout artifact cho `verify.py`. Ket qua nay chua duoc xac nhan bang run-all trong ban nhap hien tai, nhung quyet dinh ve compute la hop ly ve mat ky thuat. Neu lam lai lab ngay mai, minh se bat dau bang Colab ngay tu dau, chay NB1-NB4 truoc de co core artifacts, sau do moi quyet dinh co lam NB5/NB6 bonus hay khong. Bai hoc lon nhat la voi alignment training, kiem tra VRAM nen la buoc dau tien, truoc ca viec cai thu vien hay sua notebook.

---

## 7. Benchmark interpretation

NB6 benchmark la optional/bonus trong `verify.py`, nen ban nhap nay chua co ket qua benchmark thuc. Neu co thoi gian chay NB6, can paste/link `submission/screenshots/07-benchmark-comparison.png` va dien bang duoi day.

| Benchmark | SFT-only | SFT+DPO | Delta |
|---|---:|---:|---:|
| IFEval | TODO | TODO | TODO |
| GSM8K | TODO | TODO | TODO |
| MMLU sampled | TODO | TODO | TODO |
| AlpacaEval-lite | TODO | TODO | TODO |

Gia thuyet truoc khi chay la DPO se cai thien nhom chi so lien quan den instruction following va preference-style judgment, dac biet AlpacaEval-lite hoac cac prompt helpfulness/safety trong NB4. Tuy nhien, GSM8K va MMLU co the khong tang, thậm chi co the giam nhe, vi DPO tren preference data khong truc tiep day them suy luan toan hoc hay kien thuc factual. Neu AlpacaEval-lite tang nhung GSM8K giam, do la mot dang alignment tax: model tro nen hop voi preference hon nhung khong nhat thiet gioi hon ve reasoning. Neu MMLU giu phang, minh se xem do la tin hieu tot vi DPO adapter khong lam mat qua nhieu kien thuc nen cua SFT model.

Khi co `benchmark_results.json`, phan can doc ky nhat la delta giua SFT-only va SFT+DPO tren tung benchmark, khong phai absolute score. Vi model 3B va slice nho, absolute score co the thap va nhieu noise. Dieu dang quan tam la huong thay doi co khop voi muc tieu alignment hay khong: output duoc judge thich hon, an toan hon, nhung khong phai tra gia qua lon o reasoning/factual tasks.

---

## Bonus

- [ ] Da lam beta-sweep
- [ ] Da push len HuggingFace Hub
- [ ] Da release GGUF voi multiple quantizations
- [ ] Da link W&B run public
- [ ] Da lam cross-judge comparison
- [ ] Da lam `BONUS-CHALLENGE.md` provocation
- [ ] Pair work voi: khong

---

## Dieu ngac nhien nhat khi lam lab nay

Dieu ngac nhien nhat la DPO khong chi "train them mot adapter" nhu SFT. Ngay ca khi dung 4-bit va LoRA, memory van tang manh vi preference learning can so sanh chosen/rejected va reference behavior, nen VRAM tro thanh dieu kien quyet dinh cach lam lab.

