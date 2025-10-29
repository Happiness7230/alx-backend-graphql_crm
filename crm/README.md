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

🧩 Verify Logs
Task	Log File	Example Output
CRM Heartbeat	/tmp/crm_heartbeat_log.txt	28/10/2025-12:30:00 CRM is alive
Order Reminder	/tmp/order_reminders_log.txt	Order 1023 - customer@example.com
Low Stock Update	/tmp/low_stock_updates_log.txt	2025-10-28 - Product A (new stock: 20)
Weekly Report	/tmp/crm_report_log.txt	2025-10-27 - Report: 25 customers, 80 orders, ₦500,000 revenue

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
