DocGuard AI — Intelligent Document Forgery Detection
<div align="center">
<img src="https://img.shields.io/badge/DocGuard-AI-00e5a0?style=for-the-badge&labelColor=0a0d14" alt="DocGuard AI"/>
<img src="https://img.shields.io/badge/Status-Live%20%26%20Running-00e5a0?style=for-the-badge&labelColor=0a0d14" alt="Status"/>
<img src="https://img.shields.io/badge/Python-3.11-3776AB?style=for-the-badge&logo=python&logoColor=white" alt="Python"/>
<img src="https://img.shields.io/badge/PyTorch-2.1-EE4C2C?style=for-the-badge&logo=pytorch&logoColor=white" alt="PyTorch"/>
<img src="https://img.shields.io/badge/AWS-SageMaker-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white" alt="AWS"/>
<img src="https://img.shields.io/badge/HuggingFace-Spaces-FFD21E?style=for-the-badge&logo=huggingface&logoColor=black" alt="HuggingFace"/>
<br/><br/>
AAI-590 Capstone · University of San Diego · Shiley Marcos School of Engineering
<br/>
🚀 Live Demo  · 
📦 Model Hub  · 
🤗 HuggingFace Space
</div>
---
What is DocGuard AI?
DocGuard AI is a deployed multi-stream neural network system that detects forgery and tampering in scanned business documents. Upload any PDF, invoice, contract, or ID scan and get a forensic verdict in seconds.
> **A PDF that says 2019 but was created in 2026? DocGuard catches it with risk score 84/100 and flags: "Date mismatch — backdating detected (7yr gap)"**
Document fraud costs the global economy $4.7 trillion annually. Existing forensic tools cost $10,000+ per licence and require expert analysts. DocGuard AI is free, permanent, and accessible via a single browser upload.
---
Features
	Feature	What it catches
📅	Date mismatch	Content year vs PDF creation year — catches backdated documents
🔧	Software fingerprint	Photoshop-created PDFs, scanner anomalies, version gaps
🖼️	Pixel tampering	ELA-CNN finds photo swaps, inpainting, number changes
📝	Font inconsistency	CNN+BiLSTM detects text replacement via font switch
🗺️	Grad-CAM heatmap	Red zones on document show exactly where tampering occurred
⚠️	JavaScript in PDF	Flags PDFs with embedded JS that can modify displayed content
👤	Wiped metadata	Catches deliberately cleared author, producer, XMP fields
🔁	Double compression	Detects re-saved JPEG images hiding prior edits
---
Architecture
Three neural networks run in parallel — each detecting a different forgery signature — then fuse into one verdict:
```
Document Upload
      │
      ├── ELA-CNN (35% weight)
      │   Error Level Analysis CNN
      │   Detects: pixel tampering · photo swap · compression artifacts
      │
      ├── Metadata MLP (42% weight)
      │   18 metadata features from PDF headers
      │   Detects: date mismatch · Photoshop · wiped author · JS embedding
      │
      └── Font CRNN — CNN + BiLSTM (23% weight)
          Text line sequence analysis
          Detects: font inconsistency · text replacement

                    │
                    ▼
          Weighted Fusion Layer
                    │
          ┌─────────────────────┐
          │  🔴 HIGH RISK  70+  │
          │  🟡 MEDIUM    40-69 │
          │  🟡 LOW RISK  20-39 │
          │  🟢 AUTHENTIC  0-19 │
          └─────────────────────┘
```
---
Results
Model	Type	Test Accuracy	AUC-ROC	Converged At
ELA-CNN (from scratch)	Deep Learning	100%	1.0000	Epoch 4 / 30
Metadata MLP	Deep Learning	100%	1.0000	Epoch 10 / 50
Font CRNN — CNN+BiLSTM	Deep Learning	100%	1.0000	Epoch 2 / 10
SVM HOG+LBP baseline	Traditional ML	~100%	0.9978	—
Live system test — backdated invoice:
```
Input:   PDF dated "15 March 2019" but created as PDF in 2026

Output:  🔴 HIGH RISK — Likely Forged
         Risk Score:  84/100
         Metadata:    63/100
         Signal:      Date mismatch — backdating detected
                      Content references 2019, PDF created 2026 (7yr gap)
```
---
Quick Start
Use the live app — no setup needed
Go to https://pavanusd-docguard-ai-app.hf.space
Download `frontend/DocGuard_App.html` from this repo
Open it in your browser — connects to the permanent API automatically
Drag and drop any PDF or image
Call the API directly
```python
import requests

with open("invoice.pdf", "rb") as f:
    response = requests.post(
        "https://pavanusd-docguard-ai-app.hf.space/analyze",
        files={"file": ("invoice.pdf", f, "application/pdf")}
    )

result = response.json()
print(f"Verdict:   {result['verdict_icon']} {result['verdict']}")
print(f"Risk:      {result['forgery_risk_score']}/100")
print(f"Metadata:  {result['stream_scores']['metadata_risk']}/100")
print(f"Pixel:     {result['stream_scores']['pixel_risk']}/100")

for s in result['streams']['metadata_analysis']['signals']:
    print(f"  [{s['severity']}] {s['signal']}: {s['detail']}")
```
Run locally
```bash
git clone https://github.com/pavankallakuri9/docguardai
cd docguardai
pip install -r requirements.txt
uvicorn app:app --host 0.0.0.0 --port 8000
```
---
Repository Structure
```
docguardai/
│
├── README.md
├── app.py                           # FastAPI application (HuggingFace deployment)
├── Dockerfile                       # HuggingFace Spaces Docker config
├── requirements.txt
│
├── notebooks/
│   ├── 00_master.ipynb              # SageMaker master notebook — runs everything
│   ├── 01_train_ela_cnn.ipynb       # ELA-CNN training on Google Colab
│   └── 02_train_metadata_mlp.ipynb  # Metadata MLP + Font CRNN training
│
├── training/
│   ├── ela_cnn/train_ela_cnn.py     # SageMaker Training Job script
│   ├── metadata_mlp/train_metadata_mlp.py
│   └── font_crnn/train_font_crnn.py
│
├── inference/
│   └── inference.py                 # SageMaker inference handler
│
└── frontend/
    └── DocGuard_App.html            # Standalone web app — open and use
```
---
Infrastructure
```
Training                    Models                      Inference
──────────────────          ──────────────────────      ──────────────────────
Google Colab T4 GPU         HuggingFace Hub             HuggingFace Spaces
        ↓                   pavanusd/docguard-ai         Docker · FastAPI · CPU
Google Drive                  ela_cnn_best.pt (18.5MB)        ↓
(checkpoints)                 font_crnn_best.pt (9.7MB)  4–8s per document
        ↓                     metadata_mlp.pt (319KB)
AWS SageMaker                 metadata_scaler.pkl
eu-north-1                    metadata_features.json
ml.g4dn.xlarge GPU
15,001 files on S3
```
---
Dataset
Dataset	Source	Size	Purpose
RVL-CDIP	HuggingFace Datasets	5,000 images	Authentic class ground truth
Forgery Generator	Custom script	4,800 pairs	5 forgery types + pixel masks
Synthetic Metadata	Generated	20,000 records	18 features · 4 forgery patterns
Font Lines	Generated	10,000 images	Authentic vs mixed-font text
Five forgery types generated:
```
Text replacement    — white-out existing text, retype date/amount
Image substitution  — paste donor photo into document region
Content alteration  — correction fluid + retyped text
Double compression  — re-save at different JPEG quality
Brightness patching — regional brightness × 1.08–1.22 (scanner sim)
```
---
Team
	Name	Role	Key Contributions
🔵	Pavan Kumar Kallakuri	Lead ML Engineer & Deployment	ELA-CNN · SageMaker pipeline · HuggingFace deployment · FastAPI
🩷	Pratibha Kambi	Data Engineer & Metadata Specialist	Forgery generator · Metadata MLP · Feature engineering · SVM baseline
🟣	Sajesh Kariadan	Deep Learning & Frontend	Font CRNN · Grad-CAM · Web frontend · Report writing
---
References
Bunk et al. (2017). Detection and Localization of Image Forgeries using Resampling Features and Deep Learning. CVPRW.
Barni et al. (2020). Cover the Fingerprint: How to Forge Provenance in JPEG Images. ICASSP.
Dong et al. (2013). CASIA Image Tampering Detection Evaluation Database. ChinaSIP.
Shi et al. (2016). An End-to-End Trainable Neural Network for Image-based Sequence Recognition. IEEE TPAMI.
Selvaraju et al. (2017). Grad-CAM: Visual Explanations from Deep Networks via Gradient-based Localization. ICCV.
Harley et al. (2015). Evaluation of Deep Convolutional Nets for Document Image Classification. ICDAR.
---
<div align="center">
<img src="https://img.shields.io/badge/Made%20with-PyTorch-EE4C2C?style=flat-square&logo=pytorch&logoColor=white" alt="PyTorch"/>
<img src="https://img.shields.io/badge/Deployed%20on-HuggingFace-FFD21E?style=flat-square&logo=huggingface&logoColor=black" alt="HuggingFace"/>
<img src="https://img.shields.io/badge/Trained%20on-AWS%20SageMaker-FF9900?style=flat-square&logo=amazonaws&logoColor=white" alt="AWS"/>
<img src="https://img.shields.io/badge/USD-AAI--590%20Capstone-003DA5?style=flat-square" alt="USD"/>
<br/><br/>
Pavan Kumar Kallakuri · Pratibha Kambi · Sajesh Kariadan
<br/>
Shiley Marcos School of Engineering · University of San Diego · 2026
</div>
