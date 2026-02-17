# Product-Data-Management-project-upgrad
Developed a Python-based Product Data Management System to load, validate, update, and store product data from multiple file formats (CSV, JSON, and TXT). Implemented SKU-based validation, structured file handling, and automated data dumping while maintaining data accuracy and folder hierarchy.
# Product Data Management — Submission

**Name:** Yash Pateriya  
**Email:** yashpateriya21@gmail.com

This notebook loads CSV/JSON/TXT, updates product entries with validation, and dumps data back preserving folder structure
# Imports
import os
import json
import csv
from typing import Dict, List, Tuple
def _extract_sku_from_filename(fname: str) -> str:
    base = os.path.splitext(fname)[0]
    if '_' in base:
        sku = base.split('_')[-1]
    else:
        sku = base
    return sku

def load_data(main_folder: str) -> Tuple[Dict[str, List[int]], Dict[str, dict], Dict[str, str]]:
    sales_data = {}
    product_details = {}
    product_descriptions = {}
    # sales csv
    sales_csv_path = os.path.join(main_folder, 'sales_data.csv')
    if os.path.exists(sales_csv_path):
        with open(sales_csv_path, newline='', encoding='utf-8') as f:
            reader = csv.DictReader(f)
            for row in reader:
                sku = row.get('Product_SKU') or row.get('SKU') or row.get('product_sku')
                if not sku:
                    keys = list(row.keys())
                    sku = row[keys[0]]
                days = []
                for i in range(1,15):
                    key = f'Day{i}'
                    val = row.get(key, '')
                    try:
                        days.append(int(val))
                    except:
                        days.append(0)
                sales_data[sku] = days
    else:
        print(f"Warning: {sales_csv_path} not found. sales_data will start empty.")
    # json details
    details_folder = os.path.join(main_folder, 'product_details')
    if os.path.isdir(details_folder):
        for fname in os.listdir(details_folder):
            if fname.lower().endswith('.json'):
                path = os.path.join(details_folder, fname)
                try:
                    with open(path, 'r', encoding='utf-8') as f:
                        data = json.load(f)
                    sku = _extract_sku_from_filename(fname)
                    product_details[sku] = data
                except Exception as e:
                    print(f"Failed to load JSON {fname}: {e}")
    else:
        print(f"Warning: {details_folder} not found. product_details will start empty.")
    # txt descriptions
    desc_folder = os.path.join(main_folder, 'product_descriptions')
    if os.path.isdir(desc_folder):
        for fname in os.listdir(desc_folder):
            if fname.lower().endswith('.txt'):
                path = os.path.join(desc_folder, fname)
                try:
                    with open(path, 'r', encoding='utf-8') as f:
                        text = f.read().strip()
                    sku = _extract_sku_from_filename(fname)
                    product_descriptions[sku] = text
                except Exception as e:
                    print(f"Failed to load TXT {fname}: {e}")
    else:
        print(f"Warning: {desc_folder} not found. product_descriptions will start empty.")
    return sales_data, product_details, product_descriptions
def update_sales_data(sales_data: dict, sku: str, sales_list: List[int]) -> None:
    if not isinstance(sales_list, list) or len(sales_list) != 14:
        raise ValueError('sales_list must be a list of 14 integers')
    sales_data[sku] = [int(x) for x in sales_list]

def update_product_details(product_details: dict, sku: str, details: dict) -> None:
    product_details[sku] = details.copy()

def update_product_description(product_descriptions: dict, sku: str, description: str) -> None:
    product_descriptions[sku] = str(description)
def update(product_details: dict, sales_data: dict, product_descriptions: dict) -> Tuple[dict, dict, dict]:
    print('--- Update product (interactive) ---')
    sku = input('Enter SKU (exactly 13 characters): ').strip()
    if len(sku) != 13:
        print('Error: SKU must be exactly 13 characters long. Aborting update.')
        return product_details, sales_data, product_descriptions

    sales_input = input('Enter 14 whole numbers (sales for last 14 days) separated by spaces: ').strip()
    parts = sales_input.split()
    if len(parts) != 14:
        print('Error: You must enter exactly 14 numbers. Aborting update.')
        return product_details, sales_data, product_descriptions
    try:
        sales_list = [int(x) for x in parts]
    except Exception:
        print('Error: Sales must be whole numbers. Aborting update.')
        return product_details, sales_data, product_descriptions

    print('Enter product details:')
    name = input('  Name: ').strip()
    brand = input('  Brand: ').strip()
    model = input('  Model: ').strip()
    specifications = input('  Specifications (comma-separated recommended): ').strip()
    price = input('  Price (e.g. 1.99 or $1.99): ').strip()
    availability = input('  Availability (e.g. In stock): ').strip()

    description = input('Enter product description (text): ').strip()
    required = [name, brand, model, specifications, price, availability, description]
    if any(not x for x in required):
        print('Error: All product detail fields and description are required. Aborting update.')
        return product_details, sales_data, product_descriptions

    price_val = price
    if price_val.startswith('$'):
        price_val = price_val[1:]
    try:
        price_float = float(price_val)
    except Exception:
        price_float = price_val

    details = {
        'name': name,
        'brand': brand,
        'model': model,
        'specifications': specifications,
        'price': price_float,
        'availability': availability
    }

    update_product_details(product_details, sku, details)
    update_sales_data(sales_data, sku, sales_list)
    update_product_description(product_descriptions, sku, description)

    print(f'Success: Product {sku} added/updated.')
    return product_details, sales_data, product_descriptions
    def dump_data(sales_data: dict, product_details: dict, product_descriptions: dict, main_folder: str) -> None:
    os.makedirs(main_folder, exist_ok=True)
    details_folder = os.path.join(main_folder, 'product_details')
    desc_folder = os.path.join(main_folder, 'product_descriptions')
    os.makedirs(details_folder, exist_ok=True)
    os.makedirs(desc_folder, exist_ok=True)

    csv_path = os.path.join(main_folder, 'sales_data.csv')
    fieldnames = ['Product_SKU'] + [f'Day{i}' for i in range(1,15)]
    with open(csv_path, 'w', newline='', encoding='utf-8') as f:
        writer = csv.DictWriter(f, fieldnames=fieldnames)
        writer.writeheader()
        for sku in sorted(sales_data.keys()):
            row = {'Product_SKU': sku}
            days = sales_data.get(sku, [0]*14)
            for i, val in enumerate(days, start=1):
                row[f'Day{i}'] = val
            writer.writerow(row)

    for sku, details in product_details.items():
        fname = f'details_{sku}.json'
        path = os.path.join(details_folder, fname)
        with open(path, 'w', encoding='utf-8') as f:
            json.dump(details, f, indent=2, ensure_ascii=False)

    for sku, desc in product_descriptions.items():
        fname = f'description_{sku}.txt'
        path = os.path.join(desc_folder, fname)
        with open(path, 'w', encoding='utf-8') as f:
            f.write(str(desc))

    print(f'Data dumped to folder: {os.path.abspath(main_folder)}')
# Create sample main_folder and sample files so the notebook can be run without external files.
main_folder = 'mainfolder'  # change this path if you want to use a different folder

os.makedirs(main_folder, exist_ok=True)
os.makedirs(os.path.join(main_folder, 'product_details'), exist_ok=True)
os.makedirs(os.path.join(main_folder, 'product_descriptions'), exist_ok=True)

# Sample sales CSV
sample_sales = {
    'ABC123ABC123A': [5,2,3,4,5,6,7,8,9,10,11,12,13,14],
    'XYZ987XYZ987X': [1,1,2,2,3,3,4,4,5,5,6,6,7,7]
}
import csv
csv_path = os.path.join(main_folder, 'sales_data.csv')
with open(csv_path, 'w', newline='', encoding='utf-8') as f:
    writer = csv.writer(f)
    header = ['Product_SKU'] + [f'Day{i}' for i in range(1,15)]
    writer.writerow(header)
    for sku, days in sample_sales.items():
        writer.writerow([sku] + days)

# Sample JSON details
sample_details = {
    'ABC123ABC123A': {'name':'Sample Prod A','brand':'BrandA','model':'A1','specifications':'SpecA','price':9.99,'availability':'In stock'},
    'XYZ987XYZ987X': {'name':'Sample Prod X','brand':'BrandX','model':'X1','specifications':'SpecX','price':19.99,'availability':'Out of stock'}
}
for sku, details in sample_details.items():
    path = os.path.join(main_folder, 'product_details', f'details_{sku}.json')
    with open(path, 'w', encoding='utf-8') as f:
        json.dump(details, f, indent=2, ensure_ascii=False)

# Sample descriptions
sample_desc = {
    'ABC123ABC123A': 'This is product A. Nice item.',
    'XYZ987XYZ987X': 'This is product X. Another item.'
}
for sku, desc in sample_desc.items():
    path = os.path.join(main_folder, 'product_descriptions', f'description_{sku}.txt')
    with open(path, 'w', encoding='utf-8') as f:
        f.write(desc)

print('Sample data created in folder:', os.path.abspath(main_folder))
 Load data, add the Pokemon product, and dump back
sales_data, product_details, product_descriptions = load_data(main_folder)

# Pokemon product per assignment
sku = 'CMWKCILOP27KF'
sales_list = [8,14,16,7,15,21,14,16,32,29,26,30,25,22]
details = {
    'name': 'Pokemon Card',
    'brand': 'GameFreak',
    'model': 'ScarletViolet151',
    'specifications': 'Genuine, TCG, English',
    'price': 1.99,
    'availability': 'In stock'
}
description_text = 'Original Pokemon TCG Pikachu card'

update_sales_data(sales_data, sku, sales_list)
update_product_details(product_details, sku, details)
update_product_description(product_descriptions, sku, description_text)

# Dump to disk
dump_data(sales_data, product_details, product_descriptions, main_folder)

# Show some output
print('SKUs now in sales_data:', list(sales_data.keys()))
print('SKUs now in product_details:', list(product_details.keys()))
print('SKUs now in product_descriptions:', list(product_descriptions.keys()))
