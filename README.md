# NAICS Kod Eşleştirme Pipeline (Embedding + Cosine Similarity + LLM Hakemliği)

Bu proje, muhasebe/fatura verilerini **NAICS (North American Industry Classification System)** kodları ile otomatik olarak eşleştiren bir yapay zeka destekli pipeline sunar.  
Amaç; serbest metin (tedarikçi isimleri, ürün/hizmet açıklamaları, kısaltmalar) şeklindeki ham muhasebe verilerini anlamlı hale getirip, karbon ayak izi ve sürdürülebilirlik raporlamaları için kullanılabilir **standart sınıflara** dönüştürmektir.

**Teknik Yaklaşım:** Metin temizleme → Çok dilli embedding → Cosine similarity ile Top-K aday → LLM hakemliği (JSON) → Raporlama (Excel/JSON).

**Tam otomatik eşleştirme:** Kullanıcı “Shell motorin” yazdığında doğru NAICS kategorisine otomatik atama  
**Hibrit mimari:** Sentence-Transformers ile cosine similarity + LLM hakemliği (Top-K shortlist içinden tek seçim)  
**Çok dilli destek:** Türkçe muhasebe kayıtları İngilizce NAICS açıklamaları  
**Raporlama:** Sonuçları Excel/JSON olarak dışa aktarma; LLM gerekçesi dahil  
**Performans optimizasyonu:** 4-bit quantization (BitsAndBytes), kısa JSON çıktı

## Özellikler

**Veri Hazırlama & Temizlik:**  
  - Excel tablosundaki satırları fatura bazında birleştirir.  
  - Türkçe karakter, kısaltma, boşluk/noktalama işaretleri ve ölçü birimlerini temizler.  
  - Tekrarlayan kelime/ifadeleri kaldırır.

**Çok Dilli Embedding:**  
  - [`sentence-transformers/paraphrase-multilingual-mpnet-base-v2`](https://huggingface.co/sentence-transformers/paraphrase-multilingual-mpnet-base-v2) modeli ile metinleri vektör uzayına gömer.  
  - Türkçe + İngilizce karışık verilerde yüksek doğruluk.

**Cosine Similarity ile Aday Üretme:**  
  - Her fatura için NAICS açıklamaları ile benzerlik skoru hesaplar.  
  - Top-10 en benzer adayları çıkarır, `THRESHOLD` değeri ile güvenilir skorları otomatik seçer.

**LLM Hakemliği (Meta Llama 3.1 8B):**  
  - Embedding sonuçları yeterli değilse, LLM (Large Language Model) [meta-llama/Llama-3.1-8B-Instruct](https://huggingface.co/meta-llama/Llama-3.1-8B-Instruct) devreye girer.  
  - Fatura kalemlerini, tedarikçi bilgisini ve Top-10 aday listesini analiz ederek tek bir NAICS kodu seçer.  
  - Türkçe kısa bir gerekçe üretir (ör. “Bu fatura telekomünikasyon hizmeti olduğu için 517312 kodu seçildi.”).

**Çıktı:**  
  - Tedarikçi, Ürün/Hizmet, embedding NAICS kodu ve kategorisi, embedding skoru, Top-3 & Top-10 aday listesi, LLM tarafından seçilen NAICS kodu ve kategorisi son olarak LLM açıklaması ile detaylı Excel çıktısı üretir.

**Başarı (örnek benchmark):**  
  - Sadece embedding Top-1: %72  
  - Embedding + LLM: %86–91 (veri setine göre)  
  - Performans: 100 fatura ≈ 8–12 dk (GPU, 4-bit LLM)  



## English Summary
**NAICS Matching Pipeline (Embedding + Cosine Similarity + LLM Referee)**  
An NLP pipeline that automatically classifies accounting/invoice texts into NAICS categories and links them to the relevant emission factor.  

**Core idea:** Text cleaning → multilingual embeddings → cosine similarity (Top-K) → LLM referee (JSON) → Excel/JSON reporting.  
**Hybrid approach:** Sentence-Transformers + LLaMA-3.1-8B-Instruct  
**Multilingual:** Turkish invoices ↔ English NAICS descriptions  
**Reporting:** Top-K, scores, short Turkish reasoning  
**Perf:** 4-bit quantization (BitsAndBytes), batch encoding  

**Quick start:** install requirements → prepare cleaned inputs → run embedding match → run LLM JSON referee → export Excel.

**Benchmarks:** Embedding-only Top-1 ≈ 72% vs Hybrid ≈ 86–91%.
Why LLM as referee? Passing thousands of labels directly to an LLM is slow/costly. Top-K shortlists make it fast & accurate.
