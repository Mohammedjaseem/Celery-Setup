# Celery-Setup

  1. Inital we need to install celery
     
         Beat - pip install django-celery-beat
         Celery - pip install celery
         Results - pip install django-celery-results

            
  2. Adding celery to settings
     
         #Celery
         'django_celery_beat',
         'django_celery_results',
  
  3. Install Redis Server
  4. Create "celery.py" file in project folder
  5. Add celery related codes inside it ( here i added code for youtube downloader as example )

           
          from __future__ import absolute_import, unicode_literals
          import os
          from celery import Celery
          from django.conf import settings
          
          os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'Youtube_Downloader.settings')
          app = Celery('Youtube_Downloader')
          
          app.conf.enable_utc = False
          app.conf.update(timezone = 'Asia/Kolkata')
          app.config_from_object(settings, namespace='CELERY')
          app.autodiscover_tasks()
          
          
          def debug_task(self):
              print('f,Request:{self.request!r}')

6. Create "Tesk.py" in your APP folder
7. In task.py add the sheduled task's in this file ( Shared task example )

        @shared_task(bind=True)
        def download_video_task(self, video_url):
            task = Task()
            task.video_url = video_url
            task.status = 'STARTED'
            task.save()
    
        try:
            ydl_opts = {
                'outtmpl': f'videos/{task.id}.' + '%(ext)s'
            }
            with YoutubeDL(ydl_opts) as ydl:
                info = ydl.extract_info(video_url, download=True)
    
            task.status = 'SUCCESS'
            task.progress = 100.0
            task.error_message = "No error occured"
            task.video_link = 'videos/'+ f'{task.id}' +'.'+info['ext']
            task.save()
            return "Video downloader"
    
        except Exception as e:
            task.status = 'FAILED'
            task.progress = 0.0
            task.error_message = str(e)
            task.save()
            return "Download Failed"

8. Now in "views.py" inside APP folder ( Example here )

         #task functions start here
        def add_task(request):
            if request.method == 'POST':
                # video_url = 
                download_video_task.delay(request.POST['video_url'])
                return redirect('task_list')
            return render(request, 'add_task.html')
        
        
        def delete_task(request, task_id):
            task = Task.objects.get(id=task_id)
            task.delete()
            return redirect('task_list')
        
        def task_list(request):
            tasks = Task.objects.all()
            return render(request, 'task_list.html', {'tasks': tasks})


9. setu up urls for this function in views
10. Now we can start celery, celery beat & worker 

      To start clerey worker
      
          celery -A {{project_name}} worker --loglevel=info
      
      To start clerey beat
      
            celery -A {{project_name}} beat --scheduler django_celery_beat.schedulers:DatabaseScheduler --loglevel=info


11. Supervisor Command's
    
      To know status of processes :
          
            sudo supervisorctl status
      
      To stop celery & beat: 
          
            sudo supervisorctl stop all
      
      To restart celery & beat: 
      
          sudo supervisorctl restart all
      
      To Flush
      
          gunciorn & supervisor restart
      
      To Queue full purge 
      
          celery -A {ProjectName} purge
      
      
      Other Celery supervisor commands to enable supervisor
      
          sudo nano /etc/supervisor/conf.d/celery.conf
          sudo supervisorctl reread
          sudo supervisorctl update
          sudo supervisorctl status
          sudo supervisorctl start celery
          sudo supervisorctl start celery-beat
