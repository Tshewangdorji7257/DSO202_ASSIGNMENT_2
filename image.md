![alt text](image.png)
![alt text](image-1.png)
![alt text](image-2.png)

task created
![alt text](image-3.png)
![alt text](image-4.png)
before migration

![alt text](image-5.png)
![alt text](image-6.png)

marker task was created
![alt text](image-7.png)

Copy the old manifests

![alt text](image-8.png)

Delete the old DB Deployment
![alt text](image-9.png)

Delete the old PVC
![alt text](image-10.png)

Create statefulset.yaml
![alt text](image-11.png)

Apply the StatefulSet
![alt text](image-12.png)
![alt text](image-13.png)

new persistent storage was successfully created.
![alt text](image-14.png)

Verify the database Service
![alt text](image-15.png)

Verify the backend connection
![alt text](image-16.png)
![alt text](image-17.png)

Now test persistence
![alt text](image-18.png) StatefulSet deleted, but persistent storage remains.

Recreate the StatefulSet
![alt text](image-19.png)
![alt text](image-20.png)


Frontend migration
Install the Ingress controller
![alt text](image-21.png)
![alt text](image-22.png)

create the Ingress
![alt text](image-23.png)
![alt text](image-24.png)

Port-forward the Ingress controller

![alt text](image-25.png)
![alt text](image-26.png)
![alt text](image-27.png)

Verify the complete application
verify ingress
![alt text](image-28.png)



![alt text](image-29.png)

5. Verify the StatefulSet/PersistentVolume

![alt text](image-30.png)

2. Test backend through Ingress
![alt text](image-31.png)