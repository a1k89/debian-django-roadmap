# debian-django-roadmap
Deploy Django on Debian server step by step

Short list:
1. Connect to server via root and create new user. Add new user to sudo group
2. Setup SSH. Configure SSH. Connect to server via ssh key
3. Install must have packages (prepare)
4. Install python
5. Clone the project
6. Install and configure PostgreSQL
7. Install packages from requirements.txt. Test running via ./manage.py runserver
8. Configure nginx, gunicorn, supervisor
9. Configre static and media folder. Collect static
10. (optional) Configure SSL for domain
