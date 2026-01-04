# file-hygiene-cleaner
Automated Data Governance Suite: Audits, sanitizes, and archives corporate data for GDPR compliance and storage efficiency.

# Barkatali Data Hygiene Protocol v1.0

**Automated Data Governance & Compliance Engine**

## Overview
The Barkatali Data Hygiene Protocol v1.0 is a modular Python automation suite designed for businesses (Legal, Construction, Real Estate) to maintain data compliance and reduce liability. 

It transforms unstructured file systems into compliant, organized archives by autonomously performing risk assessments, file sanitization, and retention-based cleanup.

## Core Modules

### 1. The Auditor (`File Audit Report`)
*   **Function:** Non-destructive risk assessment.
*   **Output:** Generates CSV reports identifying "Risk Files" (e.g., passwords, credentials), ancient data (>3 years), and storage hogs.
*   **Compliance:** helps identify GDPR liabilities.

### 2. The Sanitizer (`Bulk File Renamer`)
*   **Function:** Standardization engine.
*   **Logic:** Removes illegal characters, standardizes spacing, and fixes naming conventions to ensure server compatibility.

### 3. The Cleaner (`File Cleaner`)
*   **Function:** Retention policy enforcement.
*   **Logic:** Identifies files exceeding the retention period (e.g., 30 days, 7 years) and moves them to a secure Quarantine location before deletion.

### 4. The Orchestrator (`Hygiene Runner`)
*   **Function:** The central command script.
*   **Features:** detailed logging, dry-run safety modes, and JSON-based configuration.

---

## 🛠️ Configuration
The system is controlled via `config.json`. No code changes are required for client deployment. What I would do (just to get the hard part out of the way is make the hygiene_cleaner.py script first... which is:

import sys
import os
import json
import logging
import datetime
from pathlib import Path
# HARDCODED PATHS TO FIX IMPORT ISSUES
sys.path.append(r"C:\Users\Zain\Documents\Python Work\File Cleaner")
sys.path.append(r"C:\Users\Zain\Documents\Python Work\File Audit Report")
sys.path.append(r"C:\Users\Zain\Documents\Python Work\File Renamer") 
try:
    import cleaner as file_cleaner
    import auditor as file_auditor
    import renamer as file_renamer
except ImportError as e:
    print(f"CRITICAL ERROR: Could not find your tools. {e}")
    sys.exit(1)
current_dir = Path(__file__).resolve().parent
def setup_logging(log_dir):
    Path(log_dir).mkdir(parents=True, exist_ok=True)
    timestamp = datetime.datetime.now().strftime("%Y-%m-%d")
    log_file = Path(log_dir) / f"Hygiene_Log_{timestamp}.txt"
    logging.basicConfig(filename=log_file, level=logging.INFO, format='%(asctime)s | %(levelname)s | %(message)s', datefmt='%H:%M:%S')
    console = logging.StreamHandler()
    console.setLevel(logging.INFO)
    logging.getLogger('').addHandler(console)
    return log_file
def load_config():
    config_path = current_dir / "config.json"
    if not config_path.exists():
        print(f"Error: config.json not found at {config_path}")
        sys.exit(1)
    with open(config_path, 'r') as f:
        return json.load(f)
def run_cycle():
    config = load_config()
    settings = config['settings']
    setup_logging(settings['log_location'])

Now below... is the config.json script :).
    
    logging.info("==============================")
    logging.info(f"     STARTING HYGIENE CYCLE: {config['client_name']}")
    logging.info(f"   MODE: {'DRY RUN (Safe)' if settings['dry_run'] else 'LIVE (Destructive)'}")
    logging.info("==============================")
    
    targets = config['target_directories']
    for folder in targets:
        folder_path = Path(folder)
        if not folder_path.exists():
            logging.warning(f"Skipping missing folder: {folder}")
            continue
        logging.info(f"\n>>> PROCESSING: {folder}")
        
        try:
            file_renamer.sanitize_filenames(folder, dry_run=settings['dry_run'])
        except Exception as e:
            logging.error(f"Renamer failed: {e}")
        try:
            file_auditor.generate_audit_report(folder, settings['report_location'])
        except Exception as e:
            logging.error(f"Auditor failed: {e}")
        try:
            file_cleaner.clean_directory(target_folder=folder, archive_folder=settings['archive_location'], days_old=settings['days_to_keep'], dry_run=settings['dry_run'])
        except Exception as e:
            logging.error(f"Cleaner failed: {e}")
    
    logging.info("\n========================================")
    logging.info("      CYCLE COMPLETE. HAVE A NICE DAY.")
    logging.info("========================================")
    
if __name__ == "__main__":
    run_cycle()
<img width="510" height="1362" alt="image" src="https://github.com/user-attachments/assets/30048974-e41e-478a-9f12-976b96787cb0" />


```json
{
    "client_name": "Client Name",
    "target_directories": ["C:/Server/Data"],
    "settings": {
        "dry_run": true,
        "days_to_keep": 365,
        "archive_location": "C:/Server/Quarantine"
    }
}

