# AI Resume Screening System

A Streamlit web app that ranks PDF resumes against a job description and shows the matched and missing skills for each resume. It runs on AWS EC2 and saves uploaded PDFs to Amazon S3 and scored results to Amazon DynamoDB.

> Screening scores are estimates, not measures of a candidate's potential. Do not use them as the sole basis for hiring decisions. The tool does not infer or score age, gender, religion, caste, disability, photographs or other sensitive personal attributes.

## Team

| Name | Roll No. |
|---|---|
| Parth Wankhede | 58 |
| Shashwat Deshpande | 64 |
| Soumya Dhole | 65 |
| Trigun Jagal | 66 |

## Features

- Paste a job description and upload one or more PDF resumes
- TF-IDF text similarity between each resume and the job description
- Skills coverage from an editable skill dictionary, with matched and missing skills listed
- Adjustable score weights (default 60% TF-IDF, 40% skills; always total 100%)
- Ranked results table with CSV download
- Each run saves PDFs to S3 (`resumes/<run_id>/`) and result rows to DynamoDB

## How the score works

`overall = w * TF-IDF similarity + (1 - w) * skills coverage`, with `w = 0.60` by default.

Example from a live run: 0.60 x 37.21 + 0.40 x 29.41 = 34.09.

## Architecture

```
User browser (port 8501)
        |
        v
EC2 t3.micro (ap-south-1)  --  Streamlit app.py + matcher.py
        |  (IAM role attached to the instance)
        +--> Amazon S3        PDFs saved per run
        +--> Amazon DynamoDB  table ScreeningResults (partition key run_id, sort key filename)
```

Security group: SSH (22) and Streamlit (8501).

## Project files

| File | Purpose |
|---|---|
| `app.py` | Streamlit interface, S3 and DynamoDB saving |
| `matcher.py` | Text extraction, TF-IDF similarity and skill matching |
| `requirements.txt` | Python dependencies |
| `docs/` | Project presentation (PDF) |

## Run locally

```bash
git clone https://github.com/ParthWankhede10/AI-Resume-Screening-System.git
cd AI-Resume-Screening-System
python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate
pip install -r requirements.txt
streamlit run app.py
```

Saving to S3 and DynamoDB needs AWS credentials with access to your own bucket and table. Check `app.py` for the bucket and table names and change them to yours.

## Deploy on AWS

1. Create an S3 bucket and a DynamoDB table (`ScreeningResults`, partition key `run_id`, sort key `filename`) in the same region.
2. Create an IAM role for EC2 with access to that bucket and table, and attach it to the instance.
3. Launch an EC2 instance with a security group that allows ports 22 and 8501.
4. Connect over SSH, clone this repository and install `requirements.txt`.
5. Start the app: `streamlit run app.py --server.port 8501 --server.address 0.0.0.0`
6. Open `http://<ec2-public-ip>:8501`.

## Future work

- AWS Lambda to validate uploads
- CloudWatch logs and alarms
- HTTPS with a domain name
- OCR for scanned PDFs and semantic matching
