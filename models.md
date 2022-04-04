```python
class Video(models.Model):
    video = models.FileField(upload_to='videos_uploaded',null=True,validators=[FileExtensionValidator(allowed_extensions=['MOV','avi','mp4','webm','mkv'])])
    name = models.CharField(max_length=100)
    thumbnail = models.ImageField(upload_to="video_thumbnails",null=True)
    description = models.TextField(max_length=200)
    created_by = models.ForeignKey('accounts.User', on_delete=models.CASCADE, related_name='videos_created_by')
    updated_by = models.ForeignKey('accounts.User', on_delete=models.CASCADE, related_name='videos_updated_by')

    def __str__(self):
        return self.name


class Tag(models.Model):
    name=models.CharField(max_length=100)
    def __str__(self):
        return self.name

class Category(models.Model):
    title=models.CharField(max_length=200)
    created_by = models.ForeignKey('accounts.User', on_delete=models.CASCADE, related_name='catagoery_created_by')
    updated_by = models.ForeignKey('accounts.User', on_delete=models.CASCADE, related_name='catagoery_exercise_updated_by')

    def __str__(self):
        return self.title



class Exercise(models.Model):
    name=models.CharField(max_length=100)
    description=models.TextField()
    video=models.ForeignKey(Video,on_delete=models.CASCADE)
    category=models.ForeignKey(Category,on_delete=models.CASCADE)
    tag=models.ManyToManyField(Tag)
    created_by = models.ForeignKey('accounts.User', on_delete=models.CASCADE, related_name='exercise_created_by')
    updated_by = models.ForeignKey('accounts.User', on_delete=models.CASCADE, related_name='exercise_updated_by')


    def __str__(self):
        return self.name

class Package_Exercise(models.Model):
  package = models.ForeignKey(Package,on_delete=models.CASCADE,related_name='package')
  exercise = models.ForeignKey(Exercise,on_delete=models.CASCADE)
  description=models.TextField()
  repetition = models.IntegerField()
  number_of_sets = models.IntegerField() 
  rest_time = models.IntegerField() 
  created_by = models.ForeignKey('accounts.User', on_delete=models.CASCADE, related_name='packages_exercise_created_by')
  updated_by = models.ForeignKey('accounts.User', on_delete=models.CASCADE, related_name='packages_exercise_updated_by')
    
status=(
        ("1","Pending"),
        ("2","confirm"),
        ("3","Processing"),
        ("4","Failed"),
        ("5","Done"),
    )

class Enrollment(models.Model):
    
    
    package=models.ForeignKey(Package,on_delete=models.CASCADE)
    profile=models.ForeignKey(Profile,on_delete=models.CASCADE)
    user_id=models.ForeignKey('accounts.user',on_delete=models.CASCADE)
    created_by = models.ForeignKey('accounts.User', on_delete=models.CASCADE, related_name='enrolment_created_by')
    updated_by = models.ForeignKey('accounts.User', on_delete=models.CASCADE, related_name='enrolment_updated_by')




class  Transaction(models.Model):
    user_id=models.ForeignKey('accounts.user',on_delete=models.CASCADE)
    status=models.CharField(max_length=30,choices=status)
    mode_of_payment=models.CharField(max_length=30)
    package_id=models.ForeignKey(Package,on_delete=models.CASCADE)

    # def __str__(self):
    #     return self.

class TopCard(models.Model):
    title=models.CharField(max_length=100)
    description=models.TextField()

class ExerciseLibrary(models.Model):
    image=models.ImageField(upload_to="image_thumbnails",null=True)
    title=models.CharField(max_length=300)
class CrystalClear(models.Model):
    heading=models.CharField(max_length=300)
    content=models.TextField(max_length=900)
    image=models.ImageField(upload_to="image_thumbnails",null=True)
class ScientificallyProven(models.Model):
    heading=models.CharField(max_length=300)
    content=models.TextField(max_length=1000)
class RefrencedClient(models.Model):
    content=models.TextField(max_length=500)
class Image(models.Model):
    image=models.ImageField(upload_to="image_thumbnails",null=True)

class TopImages(models.Model):
    web_image = models.ImageField(upload_to="web_thumbnails",null=True)
    mobile_image = models.ImageField(upload_to="mobile_thumbnails",null=True)
   

class Package(models.Model):
    package_name=models.CharField(max_length=100)
    doctor=models.ForeignKey(Doctor,on_delete=models.CASCADE)
    is_public=models.BooleanField(default=False)
    public_doctor=models.BooleanField(default=False)
    thumbnail= models.ImageField(upload_to="package_thumbnails",null=True)
    description=models.TextField(max_length=400)
    amount=models.FloatField()
    created_by = models.ForeignKey('accounts.User', on_delete=models.CASCADE, related_name='packages_created_by')
    updated_by = models.ForeignKey('accounts.User', on_delete=models.CASCADE, related_name='packages_updated_by')

    def __str__(self):
        return self.package_name
        


class DoctorRecommended(models.Model):
    package=models.ForeignKey('employee.Package',on_delete=models.CASCADE)
    doctor=models.ForeignKey(Doctor,on_delete=models.CASCADE)
    profile=models.ForeignKey('employee.Profile',on_delete=models.CASCADE)
    status= models.BooleanField(default=False)
