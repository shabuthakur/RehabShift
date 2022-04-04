## Doctor urls 
```python 
path('get-all-videos/', GetAllVideos.as_view()),
path('get-my-packages/',GetDoctorPackages.as_view()),
path('create-package/',DoctorCreatePackage.as_view()),
```

## Doctor APIs
```python 
class GetAllVideos(APIView):
    roles=[DOCTOR,]
    def get(self, request, *args, **kwargs):
        try:
            
            videos=Video.objects.all()
            serializer=VideoSerializer(videos,many=True)
            return Response({'status':'SUCCESS', 'data': serializer.data}, status=200)
        except Exception as e:
            return Response({'status':'ERROR', 'message': str(e)}, status=400)

class GetDoctorPackages(APIView):
    roles = [DOCTOR, ORGANISATION_ADMIN]

    def get(self, request, *args, **kwargs):
        try:
            if 'doctor_id' in request.GET:
                doctor = request.GET['doctor_id']
                package = Package.objects.filter(doctor=doctor)
            else:
                doctor = Doctor.objects.get(user=self.request.user.id)
                package = Package.objects.filter(doctor=doctor.id)
            serializer = PackageSerializer(package,many=True)
            return Response({'status':'SUCCESS', 'data': serializer.data}, status=200)    
        except Exception as e:
            return Response({'status':'ERROR', 'message': str(e)}, status=400)



class DoctorCreatePackage(APIView):
    roles = [DOCTOR,]

    def post(self, request, *args, **kwargs):
        try:
            thumbnail = request.data['thumbnail']
            print(thumbnail)
            package_data = json.loads(request.data['package'])

            print(type(package_data))
            package_data['created_by'] = self.request.user.id
            package_data['updated_by'] = self.request.user.id
            package_data['doctor'] = Doctor.objects.get(user=self.request.user).id
            package_data['thumbnail'] = thumbnail
            exercises = json.loads(request.data['exercises'])
            serializer = PackageSerializer(data=package_data)
            serializer.is_valid(raise_exception=True)
            serializer.save()
            pkg_queryset = [Package_Exercise.objects.create(package=Package.objects.get(id=serializer.data['id']),exercise=Exercise.objects.get(id=x['id']),rest_time=x['rest_time'],repetition=x['repetition']\
                ,number_of_sets=x['number_of_sets'],description=x['description'],created_by=self.request.user, updated_by=self.request.user) for x in exercises]
            pkg_serializer = PackageExerciseSerializer(pkg_queryset,many=True)
            return Response(
                {
                    "status":"SUCCESS",
                    "data":{
                        "package": serializer.data,
                        "exercises": pkg_serializer.data
                    }
                }
                ,status=200)
        except Exception as e:
            traceback.print_exc()
            return Response({"status":"ERROR","message":str(e)},status=400)

```

## Employee urls

```python 
path('get-all-packages/',GetAllPackagesAPI.as_view()),
path('package-details/', GetPackageById.as_view()),
path('package-exercises/', GetExerciseByPackageId.as_view()),
path("get-all-exercices-for-web-withouttoken/",GetALLExerciseWithoutToken.as_view()),
path('get-banner-images/',GetBannerImages.as_view()),
path('get-Category/',GetAllCategoryAPI.as_view()),
path('get-random-packages/', GetRandomPackages.as_view()),
path('get-packages-according-to-category/',GetPackagesAccordingToCategory.as_view()),
path("get-Recommended-packages/",GetRecommendedPackagesbyDoctor.as_view()),
path('package-with-exercises/', GetPackagesWithExercises.as_view()),
path('create-package/', DoctorCreatePackage.as_view()),
path('create-enrollment/',CreateEnrollment.as_view()),
path('enrollment-check/',GetUserEnrollmentCheck.as_view()),
path('get-exercise-by-id/',GetExerciseById.as_view()),
path('my-packages/',GetMyPackages.as_view()),
path('get-public-packages/',GetPublicPackages.as_view())
```


## Employee APIs
```python 
class GetAllPackagesAPI(APIView):
    role=[DOCTOR,EMPLOYEE]
    def get(self, request, *args, **kwargs):
        try:
            
            package=Package.objects.all()
            
            serializer=PackageSerializer(package,many=True)
            return Response({'status':'SUCCESS', 'data': serializer.data}, status=200)
        except Exception as e:
            return Response({'status':'ERROR', 'message': str(e)}, status=400)
 
class GetPackageById(APIView):
    role=[EMPLOYEE]
    def get(self,request,*args,**kwargs):
        try:
            package_id=request.GET['package_id']
            packages=Package.objects.get(id=package_id)
            serializer=PackageSerializer(packages)  
            return Response({'status':'SUCCESS','data':serializer.data},status=200)
        except Exception as e:
            return Response({'status':'ERROR','message':str(e)},status=400)
class GetExerciseByPackageId(APIView):
    roles=[EMPLOYEE, ]
    def get(self,request,*args,**kwargs):
        try:
            package=request.GET['package']
            packages=Package_Exercise.objects.filter(package=package)
            serializers=PackageExerciseSerializer(packages,many=True)
            return Response({'status':'SUCCESS','package_name':str(packages.last().package),'data':serializers.data},status=200)
        except Exception as e:
            return Response({'status':'ERROR','message':str(e)},status=400)


class GetEmployeeReportsOnKiosk(ListAPIView):
    roles = [KIOSK_ADMIN]
    permission_classes = [Simple]
    def get(self, request, *argfs, **kwargs):
        try:
            print(request.user)
            profile_id = request.GET['profile_id']
            queryset = EmployeeLabReport.objects.filter(profile__id = profile_id)
            page = self.paginate_queryset(queryset)
            if page is not None:
                serializer = EmployeeLabReportsSerializer(page, many=True)
                return self.get_paginated_response(serializer.data)
            serializer = EmployeeLabReportsSerializer(queryset, many = True)
            return Response({'status':'SUCCESS', 'data': serializer.data}, status=200)
        except Exception as e:
            return Response({'status':'ERROR', 'message': str(e)}, status=400) 


class GetEmployeeLabreportImage(APIView):
    roles = [EMPLOYEE]
    def get(self, request, *args, **kwargs):
        try:
            profile_id = request.GET['profile_id']
            report_id = request.GET['report_id']
            downlodable = request.GET.get('downlodable', False)
            profile_obj = Profile.objects.get(id=profile_id)
            report_obj = EmployeeLabReport.objects.get(id=report_id, profile=profile_obj)
            if report_obj.report:
                report_url = report_obj.report.url
                file_url = 'media/'+(report_url.split('/media/'))[1]
                if downlodable:
                    return send_from_s3(file_url)
                else:
                    s3_url = send_from_s3(file_url, True)
                    return Response({'status':'SUCCESS', 'data': s3_url}, status=200)
            else:
                return Response({'status':'SUCCESS', 'data': "No file found in this report."}, status=200)
        except Exception as e:
            return Response({'status': 'ERROR', 'message': str(e)}, status=400)

class GetALLExerciseWithoutToken(APIView):
    permission_classes=[AllowAny]
    def get(self,request,*args,**kwargs):
        try:
            resultant=[]
            result={}
            sample=[]
            topcard=[]
            exercise_library=[]
            category = Category.objects.all()
            topcards=TopCard.objects.all()
            for i in topcards:
                topcard.append({
                    "title":str(i.title),
                    "description":str(i.description)
                })
            resultant.append(topcard)
            exercise=ExerciseLibrary.objects.all()
            for i in exercise:
                exercise_library.append({
                    "image":str(i.image.url),
                    "title":str(i.title)
                    })
            resultant.append(exercise_library)
            Crystal_Clear=[]
            crystal=CrystalClear.objects.all()
            for i in crystal:
                Crystal_Clear.append({
                    "heading":str(i.heading),
                    "content":str(i.content),
                    "image":str(i.image.url),
                })
            resultant.append(Crystal_Clear)
            for i in category:
                result['category']=str(i.title)
                exercises = Exercise.objects.filter(category_id=i.id)
                exer = []
                for j in exercises:
                    exer.append({
                        "name":str(j.name),
                        "video":str(j.video.video.url),
                        "description":str(j.description)

                    })
                result['Data']=exer
                sample.append(result.copy())
            resultant.append(sample)
            # # print(resultant)  
            proven=ScientificallyProven.objects.all()
            proven_list=[]
            for i in proven:
                proven_list.append({
                    "heading":str(i.heading),
                    "content":str(i.content)
                })
            resultant.append(proven_list)
            refrence_list=[]
            refrence=RefrencedClient.objects.all()
            for i in refrence:
                refrence_list.append({
                    "content":str(i.content)
                })
            resultant.append(refrence_list)
            images_list=[]
            images=Image.objects.all()
            for i in images:
                images_list.append({
                    "image":str(i.image.url)
                })
            resultant.append(images_list)
            return Response({"status":"Success","data":resultant},status=200)
        except Exception as e:
            return Response({'status':'ERROR','message':str(e)},status=400)

                
class GetBannerImages(ListAPIView):
    roles=[EMPLOYEE]
    def get(self,request, *args,**kwargs):
        try:
            type=request.GET['type']
            images=TopImages.objects.all()
            
            page = self.paginate_queryset(images)
            if type=="mobile":
                if page is not None:
                    serializer=MobileSerializer(page,many=True)
                    return self.get_paginated_response(serializer.data)
                # return Response({'status':'SUCCESS', 'data':serializer.data}, status=200)
            if type=="web":
                if page is not None:
                    serializer=WebSerializer(page,many=True)
                    return self.get_paginated_response(serializer.data)
        except Exception as e:
            return Response({'status': 'ERROR', 'message': str(e)}, status=400)

class GetAllCategoryAPI(APIView):
    roles=[EMPLOYEE]
    def get(self, request, *args, **kwargs):
        try:
            category=Category.objects.all()
            serializers=CategorySerializer(category,many=True)
            return Response({'status':'SUCCESS', 'data':serializers.data}, status=200)
        except Exception as e:
            return Response({'status': 'ERROR', 'message': str(e)}, status=400)
class GetRandomPackages(APIView):
    roles=[EMPLOYEE]
    def get(self, request, *args, **kwargs):
        try:
            packages=Package.objects.all()
            list1=[]
            # print(packages.count())
            list=random.sample(range(0,packages.count()),6)
            for i in list:
                list1.append(packages[i])
            serializers=PackageSerializer1(list1,many=True)
            return Response({'status':'SUCCESS', 'data':serializers.data}, status=200)
        except Exception as e:
            return Response({'status': 'ERROR', 'message': str(e)}, status=400)


class GetPackagesAccordingToCategory(APIView):
    roles=[EMPLOYEE]
    def get(self,request,*args,**kwargs):
        try:
            category=request.GET['category']
            exercises=Exercise.objects.filter(category=category)    
            package_exercises=Package_Exercise.objects.filter(exercise__id__in=exercises)
            serializers=PackagesAccordingToCategorySerializer(package_exercises,many=True)
            return Response({'status':'SUCCESS','data':serializers.data},status=200)
        except Exception as e:
            return Response({'status':'ERROR','message':str(e)},status=400)

class GetRecommendedPackagesbyDoctor(APIView):
    roles = [EMPLOYEE]
    def get(self, request, *args, **kwargs):
        try:
            profile_id=request.GET['profile_id']
            prf_obj = Profile.objects.get(id=profile_id)
            profiles = Profile.objects.filter(user=self.request.user.id)
            recommended=DoctorRecommended.objects.filter(profile=profile_id)
            if prf_obj in profiles:
                serializer=RecommendedPackagesbyDoctorSerializer(recommended,many=True)
                return Response({"status":"Success","data":serializer.data},status=200)
            return Response({'status':'ERROR', 'message': 'Not Recommended'}, status=400)
        except Exception as e:
            return Response({'status':'ERROR','message':str(e)},status=400)
 
class GetPackagesWithExercises(APIView):
    roles = [EMPLOYEE,]
    def get(self, request, *args, **kwargs):
        try:
            package = request.GET['package']
            package_exercise = Package_Exercise.objects.filter(package=package)
            if len(package_exercise):
                package= Package.objects.get(id=package)
                packageSerializer = PackageSerializer(package)
                exerciseSerializer = PackageExerciseSerializer(package_exercise,many=True)
                data ={
                    "package":packageSerializer.data,
                    "exercises":exerciseSerializer.data
                }
                return Response({"status":"Success","data":data},status=200)
            pack = Package.objects.get(id=package)
            serializer = PackageSerializer(pack)
            data ={
                    "package":serializer.data,
                    "exercises": None
                }
            return Response({"status":"Success","data":data},status=200)
        except Exception as e:
            return Response({"status":"Error","message":str(e)},status=400) 

class GetUserEnrollmentCheck(APIView):
    role=[EMPLOYEE, ]
    def get(self,request,*args,**kwargs):
        try: 
            profile=request.GET['profile']
            enroll=Enrollment.objects.filter(user_id=self.request.user,profile=profile)
            prf_obj = Profile.objects.get(id=profile)
            profiles = Profile.objects.filter(user=self.request.user.id)
            if prf_obj in profiles:
                serializers=EnrollmentSerializer(enroll,many=True)
                return Response({'status':'SUCCESS','data':serializers.data}, status=200)
            return Response({'status':'Error'},status=200)
        except:
            return Response({'status':'ERROR', 'message': 'User Not Subscribed'}, status=400)
 
class CreateEnrollment(APIView):
    roles=[EMPLOYEE, ]
    def post(self,request,*args,**kwargs):
        try:
            package=request.data['package']
            profile=request.data['profile']
            prf_obj = Profile.objects.get(id=profile)
            profiles = Profile.objects.filter(user=self.request.user.id)
            enrollmentcheck=Enrollment.objects.filter(profile=profile,package=package)
            if len(enrollmentcheck) != 0:
                return Response({"status":'Error','message':'this is package already exist'},status=400)
            elif prf_obj in profiles:
                Enrollment.objects.create(user_id=self.request.user,profile=Profile.objects.get(id=profile),package=Package.objects.get(id=package),created_by=self.request.user,updated_by=self.request.user)
                return Response({'status':'SUCCESS','message':'Data Created'},status=200)    
            else:
                return Response({"status":'Error','message':'Not a profile in this user'},status=400)

        except Exception as e:
                return Response({'status':'ERROR', 'message': str(e)}, status=400)

class GetExerciseById(APIView):
    roles=[EMPLOYEE, ]
    def get(self,request,*args,**kwargs):
        try:
            exercise_id=request.GET["exercise_id"]
            exercise=Exercise.objects.get(id=exercise_id)
            serializer=ExerciseSerializer(exercise)
            return Response({'status':'SUCCESS','data':serializer.data},status=200)
        except Exception as e:
            return Response({'status':'ERROR','message':str(e)},status=400)        
   
class GetMyPackages(APIView):
    roles = [EMPLOYEE]

    def get(self, request, *args, **kwargs):
        try:
            enrolled = Enrollment.objects.filter(user_id= self.request.user.id)
            package = [x.package for x in enrolled]
            serializer = PackageSerializer(package, many=True)
            return Response({"status":"Success","data":serializer.data},status=200)
        except Exception as e :
            return Response({"status":"Error","message":str(e)},status=400)

class GetPublicPackages(APIView):
    roles=[EMPLOYEE]
    def get(self,request,*args,**kwargs):
        try: 
            packages=Package.objects.filter(is_public=True)
            serializer=PackageSerializer(packages,many=True)
            return Response({"status":"Success","data":serializer.data},status=200)
        except Exception as e :
            return Response({"status":"Error","message":str(e)},status=400)

```