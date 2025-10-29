🧾 CRM Automation & GraphQL Integration
📖 Overview

This project extends a Django-based CRM system with:

GraphQL API for querying and mutations

Cron jobs (via django-crontab) for automated system checks and inventory updates

Celery + Celery Beat for scheduled background tasks

Redis as a Celery message broker

Together, these features enable automation of:

Order reminders for recent orders

Heartbeat checks for CRM uptime

Automatic stock restocking for low-inventory products

Weekly CRM performance reporting (customers, orders, and revenue)

🧩 Features Summary
Feature	Description	Schedule
Order Reminders	Logs pending orders from the past week	Daily @ 8:00 AM
CRM Heartbeat	Logs “CRM is alive” every 5 minutes	Every 5 mins
Low Stock Restock	Automatically restocks products with stock < 10	Every 12 hours
Weekly CRM Report	Logs total customers, orders, and revenue	Every Monday @ 6:00 AM

⚙️ 1. Installation
Prerequisites

Python 3.8+

Redis server running locally

Django project initialized

Install Dependencies

Activate your virtual environment, then install:

pip install django gql requests celery django-celery-beat django-crontab redis


Add to requirements.txt:

django
gql
requests
celery
django-celery-beat
django-crontab
redis

🧠 2. Project Configuration
a. Add Installed Apps

In crm/settings.py:

INSTALLED_APPS = [
    # Django defaults...
    'django_crontab',
    'django_celery_beat',
    'crm',
]

b. Default AutoField

Prevent model warnings by adding:

DEFAULT_AUTO_FIELD = 'django.db.models.BigAutoField'

c. Redis Configuration (for Celery)
CELERY_BROKER_URL = 'redis://localhost:6379/0'
CELERY_RESULT_BACKEND = 'redis://localhost:6379/0'

🧾 3. GraphQL Integration
GraphQL Endpoint

Make sure your project has a running GraphQL endpoint at:

http://localhost:8000/graphql


This endpoint exposes queries and mutations used by cron jobs and Celery tasks.

Sample Schema Additions (crm/schema.py)
a. UpdateLowStockProducts Mutation
import graphene
from .models import Product

class UpdateLowStockProducts(graphene.Mutation):
    success = graphene.Boolean()
    updated_products = graphene.List(graphene.String)

    def mutate(self, info):
        updated = []
        low_stock = Product.objects.filter(stock__lt=10)
        for product in low_stock:
            product.stock += 10
            product.save()
            updated.append(f"{product.name} (new stock: {product.stock})")
        return UpdateLowStockProducts(success=True, updated_products=updated)

class Mutation(graphene.ObjectType):
    update_low_stock_products = UpdateLowStockProducts.Field()

🕒 4. Cron Jobs (via django-crontab)
a. Setup

Add to crm/settings.py:

CRONJOBS = [
    ('*/5 * * * *', 'crm.cron.log_crm_heartbeat'),
    ('0 */12 * * *', 'crm.cron.update_low_stock'),
]

b. Cron Job Implementations (crm/cron.py)
1. Heartbeat Job
from datetime import datetime

def log_crm_heartbeat():
    timestamp = datetime.now().strftime('%d/%m/%Y-%H:%M:%S')
    message = f"{timestamp} CRM is alive\n"
    with open('/tmp/crm_heartbeat_log.txt', 'a') as log:
        log.write(message)

2. Update Low Stock Job
from gql import gql, Client
from gql.transport.requests import RequestsHTTPTransport

def update_low_stock():
    transport = RequestsHTTPTransport(url='http://localhost:8000/graphql', verify=True, retries=3)
    client = Client(transport=transport, fetch_schema_from_transport=True)

    mutation = gql('''
        mutation {
            updateLowStockProducts {
                success
                updatedProducts
            }
        }
    ''')

    result = client.execute(mutation)
    updates = result["updateLowStockProducts"]["updatedProducts"]

    from datetime import datetime
    timestamp = datetime.now().strftime('%Y-%m-%d %H:%M:%S')

    with open('/tmp/low_stock_updates_log.txt', 'a') as f:
        for product in updates:
            f.write(f"{timestamp} - {product}\n")

🧩 5. Celery + Celery Beat Setup
a. Create crm/celery.py
from __future__ import absolute_import, unicode_literals
import os
from celery import Celery
from celery.schedules import crontab

os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'crm.settings')

app = Celery('crm')
app.config_from_object('django.conf:settings', namespace='CELERY')
app.autodiscover_tasks()

b. Update crm/__init__.py
from .celery import app as celery_app

__all__ = ('celery_app',)

c. Celery Beat Schedule (in crm/settings.py)
from celery.schedules import crontab

CELERY_BEAT_SCHEDULE = {
    'generate-crm-report': {
        'task': 'crm.tasks.generate_crm_report',
        'schedule': crontab(day_of_week='mon', hour=6, minute=0),
    },
}

d. Weekly CRM Report Task (crm/tasks.py)
from celery import shared_task
from gql import gql, Client
from gql.transport.requests import RequestsHTTPTransport
from datetime import datetime

@shared_task
def generate_crm_report():
    transport = RequestsHTTPTransport(url='http://localhost:8000/graphql', verify=True, retries=3)
    client = Client(transport=transport, fetch_schema_from_transport=True)

    query = gql('''
        query {
            totalCustomers
            totalOrders
            totalRevenue
        }
    ''')

    result = client.execute(query)
    customers = result['totalCustomers']
    orders = result['totalOrders']
    revenue = result['totalRevenue']

    timestamp = datetime.now().strftime('%Y-%m-%d %H:%M:%S')
    with open('/tmp/crm_report_log.txt', 'a') as log:
        log.write(f"{timestamp} - Report: {customers} customers, {orders} orders, {revenue} revenue\n")

    print("CRM weekly report generated!")

🧰 6. Redis Setup

Install and enable Redis:

sudo apt install redis-server -y
sudo systemctl start redis-server
sudo systemctl enable redis-server


Check status:

redis-cli ping
# Expected output: PONG

🧾 7. Running the System
Run Migrations
python manage.py migrate

Run Django Server
python manage.py runserver

Start Celery Worker
celery -A crm worker -l info

Start Celery Beat
celery -A crm beat -l info

Register Cron Jobs
python manage.py crontab add
python manage.py crontab show

🧩 8. Verify Logs
Task	Log File	Example Output
CRM Heartbeat	/tmp/crm_heartbeat_log.txt	28/10/2025-12:30:00 CRM is alive
Order Reminder	/tmp/order_reminders_log.txt	Order 1023 - customer@example.com
Low Stock Update	/tmp/low_stock_updates_log.txt	2025-10-28 - Product A (new stock: 20)
Weekly Report	/tmp/crm_report_log.txt	2025-10-27 - Report: 25 customers, 80 orders, ₦500,000 revenue

9. .gitignore Recommendations

Create .gitignore in the project root:

# Python
__pycache__/
*.pyc
*.pyo
*.pyd
*.log
*.sqlite3
.env
venv/
.env/
*.egg-info/

# Celery / Redis
celerybeat-schedule
celerybeat.pid

# System logs
/tmp/*.txt
*.pid

# Django
db.sqlite3
/static/
media/

✅ Summary

This CRM system now:

Integrates GraphQL queries and mutations

Automates tasks using django-crontab and Celery Beat

Uses Redis as a distributed message broker

Generates real-time and scheduled reports

📍 Automation Timeline:

Every 5 mins → Heartbeat check

Every 12 hrs → Low stock restock

Every day @ 8 AM → Order reminder

Every Monday @ 6 AM → Weekly report
