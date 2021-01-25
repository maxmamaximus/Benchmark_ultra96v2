# User guide : Setting hardware before Benchmark Ultra96 spec on ZCU104
เอกสารนี้จะบอกเกี่ยวกับไฟล์ที่ต้องใช้งาน การจัดการไฟล์ที่จะนำมา compile เพื่อนำไปใช้บนบอร์ดหลังทำการ quantize AI model โดยเนื้อหาอย่างอื่นสามารถอ่านได้ใน [Xilinx/Vitis-AI](https://github.com/Xilinx/Vitis-AI) และ [ug1414](https://www.xilinx.com/support/documentation/sw_manuals/vitis_ai/1_0/ug1414-vitis-ai.pdf) จะแบ่งขั้นตอนการนำไปใช้ดังนี้ 

# Objective this work 

Profiling use on debugging [Hardware & package linux path of AI application]
Test AI dual core #1 

- Mission get largest 1 model from 2 core [resource] 
- Example 2 model on 1 core testing
- Clear Model on each core
# STEP 1 : การนำไฟล์ที่เตรียมไว้ไปใช้งาน

ครั้งแรกไฟล์ที่เตรียมไว้จะมีทั้งหมด 16 แบบประกอบไปด้วย arch of dpu 1152 1600 2304 ที่ความถี่ 100 150 200 300 ในแต่ละรุ่นโดยมีให้ทดสอบเฉพาะ 2 core เพื่อนำไปใช้ทดสอบดังรูป 

![](https://paper-attachments.dropbox.com/s_49F8656931A3E6AB0DFDD30FE17E245F04634FF25E3B671DA9A8CDA41D1035D2_1609925169839_image.png)


แต่เนื่องจากการทดสอบทาง Resource ([+Technical Report : Resource Ultra96V2](https://paper.dropbox.com/doc/Technical-Report-Resource-Ultra96V2-QruTE4RDbYPFotJxchAwW) )
นั้นทำให้ทราบว่าสถาปัตยกรรมที่ใช้ได้มีเพียง arch of dpu รุ่น 800 ลงไป จึงเหลือสถาปัตยกรรมให้ทดสอบดังนี้

![](https://paper-attachments.dropbox.com/s_49F8656931A3E6AB0DFDD30FE17E245F04634FF25E3B671DA9A8CDA41D1035D2_1611309792614_image.png)


โดยแต่ละชื่อแต่ละไดเรคทอรี่จะบอกถึง {arch of dpu}_{core}_{dpu frequency} 
ภายในแต่ละโฟลเดอร์จะประกอบไปด้วยไดเรคทอรี่ DPU-TRD,modelzoo,sd_card,root 
โดยสิ่งที่ผู้พัฒนา AI ต้องโฟกัสคือ modelzoo, sd_card, root

- modelzoo ข้างในจะประกอบด้วยไฟล์ที่จะใช้ในการ compile 
- sd_card ไดเรคทอรี่นี้ใช้สำหรับการเปิดบอร์ดและการทำงานรวมถึงไดรเวอร์หรือไฟล์ที่จะใช้จะถูกเตรียมไว้ในนี้ ซึ่งจะอยู่ใน partition แรกของ SD card จะมีไฟล์ zcu104_base.dcf อยู่ในไดเรคทอรี่นั้นๆ  
- root ไฟล์ system ของบอร์ดซึ่งจะต้องแตกไฟล์ไปอยู่ใน partition ที่สองของ SD card

โดยการจะนำไฟล์ไปใช้งานมีดังนี้

1. เข้าไปที่เครื่อง Cooler master
    $ ssh embedded@10.1.1.166 #password : eslab
2. เข้าไปที่ไดเรคทอรี่ 
    $ cd ~/titan/Vitis-AI-Project/source/software/Vitis-AI/ultra96v2_env_zcu104
----------
# STEP 2 : ทดสอบการ Compile AI model 

ในส่วนนี้จะทำการเตรียม model ที่ใช้งานกับ arch of dpu ที่เรากำหนดไว้ในส่วนแรก ซึ่งจะทำการ compile model AI มาใช้งานจากที่ Xilinx เตรียมให้ ซึ่งถ้าต้องการปรับแต่่ง จะสามารถศึกษาได้จาก 
  https://www.xilinx.com/support/documentation/sw_manuals/vitis_ai/1_0/ug1414-vitis-ai.pdf
  https://www.xilinx.com/support/documentation/sw_manuals/ai_inference/v1_6/ug1327-dnndk-user-guide.pdf
  [Vitis-AI 1.1 Flow for Avnet VITIS Platforms - Part 1 - Hackster.io](https://www.hackster.io/AlbertaBeef/vitis-ai-1-1-flow-for-avnet-vitis-platforms-part-1-007b0e)

![](https://hackster.imgix.net/uploads/attachments/1106858/image_psuiXfEt8s.png?auto=compress%2Cformat&w=740&h=555&fit=max)


จากรูปที่เห็นจะบอกว่า model ไหนใช้กับ application ตัวไหนบ้างเพื่อไม่ให้เกิดข้อผิดพลาดในการใช้งาน

1. หลังจากนั้นจะทำการรัน docker เพื่อใช้งาน vitis AI platform ในการ train และ compile โมเดล
    $ cd Vitis_AI
    $ ./docker_run.sh xilinx/vitis-ai:latest

#####################################################################
C**aution**

    กรณีที่มีปัญหาไม่สามารถใช้งานคำสั่งนี้ได้ ให้ตรวจสอบว่า docker ในเครื่องได้ทำการติดตั้งหรือยัง ถ้ายังให้ไปติดตั้งโดยสามารถหากลับไปติดตั้งตาม [Step 1](https://www.dropbox.com/scl/fi/7f9g25f7oha16x62gjv2k/LAB-02-_-Develop-AI-for-use-with-FPGA.paper?dl=0&rlkey=3ln9nmsbhx8accrv4bg7useg3#:uid=407525900318242205111737&h2=Part-1---Install-Docker-and-Pr) 

#####################################################################

2. หลังจากนั้นจะเข้าสู่การเตรียม caffe model ให้สามารถใช้งานบน dpu ได้
    $ conda activate vitis-ai-caffe
    (vitis-ai-caffe) $
3. นำไฟล์ hwh มาไว้ใน directory ที่สร้างใหม่เพื่อง่ายต่อการเรียกใช้และเพิ่มความสะดวกในการทำงาน
    (vitis-ai-caffe) $ cd modelzoo
4. ใช้โมดูล dlet เพื่อทำไฟล์ .dcf ที่จะใช้ในการคอมไฟล์โมเดล
    (vitis-ai-caffe) $ cp ../sd_card/{platform}.hwh .
    (vitis-ai-caffe) $ dlet -f {platform}.hwh 
5. ทำการเปลี่ยนชื่อจาก dpu(ตามด้วยวันที่ make ).dcf เป็นชื่อของ platform หรือ arch of dpu ที่ใช้ เพื่อไม่ให้เกิดความสับสน
    (vitis-ai-caffe) $ mv dpu*.dcf {platform}.dcf
6. สร้างไฟล์ .json มาเพื่อกำหนด Module และ core cpu ที่ใช้ รวมถึง platform ที่จะใช้ 
    $ vi custom.json
    {"target": "DPUCZDX8G", "dcf": "./{platform}.dcf", "cpu_arch": "arm64"}
7. หลังจากนั้นทำไฟล์ script เพื่อให้สะดวกในการคอมไพล์ แค่ add ชื่อไฟล์ argument ไม่กี่ตัวแค่แก้รับตัวแปร argument ไม่กี่ตัวเวลาคอมไพล์ไม่ต้องแก้ไฟล์ compile ก่อนรันในทุกๆครั้ง
    $ vi compile_cf_model.sh


    model_name=$1
    modelzoo_name=$2
    vai_c_caffe \
    --prototxt ../../AI-Model-Zoo/model/caffe/${modelzoo_name}/quantized/Edge/deploy.prototxt \
    --caffemodel ../../AI-Model-Zoo/model/caffe/${modelzoo_name}/quantized/Edge/deploy.caffemodel \
    --arch ./custom.json \
    --output_dir ./compiled_output/${modelzoo_name} \
    --net_name ${model_name} \
    --options "{'mode': 'normal'}"

####################################################################
**Caution**

    เช็ค path ที่เรียกให้ดีก่อนทำไฟล์ script

####################################################################

8.  จากนั้นสร้างโฟลเดอร์เพื่อเก็บไฟล์ output ที่ compile 
    (vitis-ai-caffe) $ mkdir compiled_output
9. ทำการ compile model resnet50
    (vitis-ai-caffe) $ source ./compile_cf_model.sh resnet50 cf_resnet50_imagenet_224_224_7.7G_1.2
    (vitis-ai-caffe) $ source ./compile_cf_model.sh yolov3_adas_pruned_0_9 dk_yolov3_cityscapes_256_512_0.9_5.46G_1.2 
10. ทำการตรวจสอบไฟล์ที่คอมไพล์ในไดเรคทอรี่ที่ตั้งไว้
    (vitis-ai-caffe) $ tree
    ├── compiled_output
    │   ├── cf_densebox_wider_360_640_1.11G
    │   │   ├── densebox_kernel_graph.gv
    │   │   └── dpu_densebox.elf
    │   ├── cf_facerec-resnet20_112_96_3.5G
    │       ├── dpu_facerec-resnet20.elf
    │       └── segmentation_kernel_graph.gv
    │
    ├── compile_model.sh
    ├── custom.json
    ├── {platform}.dcf
    └── {platform}.hwh
11. ออกจาก vitis too 
    (vitis-ai-caffe) $ exit
----------
# STEP 3 : การติดตั้ง environment บน ZCU104 ก่อนทดสอบ AI application
## STEP 3.1 : การเตรียม SD card
1. ทำการเตีรยม sd card ก่อนใช้งาน 
    1. นำ sd_card ที่ได้มาเสียบเข้ากับเครื่องผ่านตัวอ่าน
    2. พิมคำสั่ง
    sudo gparted
    c. หลังจากข้าโปรแกรมคลิกเลือกที่พาร์ติชัน SD card อย่างในตัวอย่างเป็น /dev/sda (เช็คจากขนาด sd_card ถ้าผิด OS อาจจะหาย)
![](https://paper-attachments.dropbox.com/s_49F8656931A3E6AB0DFDD30FE17E245F04634FF25E3B671DA9A8CDA41D1035D2_1610083719446_Screenshot+from+2021-01-08+12-28-08.png)

    d. ทำการ unmount partition SD card จากในรูปเป็น BOOT และ rootfs โดยการลิกขวาที่ partition ที่เลือก → คลิกที่ Unmount
    e. หลังจาก Unmount ให้คลิกที่สัญลักษณ์ที่แทบบนซ้ายดังรูป แล้วทำการลบทุก patition จนเหลือแค่ Unknow
    
![](https://paper-attachments.dropbox.com/s_49F8656931A3E6AB0DFDD30FE17E245F04634FF25E3B671DA9A8CDA41D1035D2_1610959098557_image.png)
![](https://paper-attachments.dropbox.com/s_49F8656931A3E6AB0DFDD30FE17E245F04634FF25E3B671DA9A8CDA41D1035D2_1610959271611_Screenshot+from+2021-01-18+15-40-36.png)

    f. ทำการสร้าง partiton มาสอง partition ดังรูป คือ boot และ rootfs โดยที่
        1. boot ต้องเป็น fat32 และมี space เผื่อไว้ 4 mib
![](https://paper-attachments.dropbox.com/s_49F8656931A3E6AB0DFDD30FE17E245F04634FF25E3B671DA9A8CDA41D1035D2_1610959308647_Screenshot+from+2021-01-18+15-41-39.png)

        ii. rootfs เป็น ext4 เป็นพื้อนที่ที่เหลือ
![](https://paper-attachments.dropbox.com/s_49F8656931A3E6AB0DFDD30FE17E245F04634FF25E3B671DA9A8CDA41D1035D2_1610959400820_Screenshot+from+2021-01-18+15-43-11.png)

    h. ทำการ validate sd card โดยคลิกที่สัญลักษณ์เครื่องหมายถูกดังรูป และรอจนเสร็จ
![ไม่มีคำอธิบาย](https://scontent.fbkk14-1.fna.fbcdn.net/v/t1.15752-9/139739134_203992791438110_3137093115371775114_n.jpg?_nc_cat=100&ccb=2&_nc_sid=ae9488&_nc_eui2=AeG3nuEfYMO1NYymKZSLHZM6_f9CY_JupGX9_0Jj8m6kZfhQZBg7zY0XCTT6Nf385UHvjez8mtmVNITG-z9go14k&_nc_ohc=pTcM990ATxoAX9gsTNy&_nc_ht=scontent.fbkk14-1.fna&oh=3bd1b5aba30cae6aa5caada9eaf9143b&oe=602AACA1)

    i. รอจนเสร็จการ validate ดังทั้ง สองรูป มีทั้งก่อนและหลัง รอจนเสร็จแล้วค่อยปิดโปรแกรม เราจะได้ SD card เปล่ามาใช้งาน
    
![](https://paper-attachments.dropbox.com/s_49F8656931A3E6AB0DFDD30FE17E245F04634FF25E3B671DA9A8CDA41D1035D2_1610959899656_Screenshot+from+2021-01-18+15-46-18.png)
![](https://paper-attachments.dropbox.com/s_49F8656931A3E6AB0DFDD30FE17E245F04634FF25E3B671DA9A8CDA41D1035D2_1610959899650_Screenshot+from+2021-01-18+15-46-26.png)

2. ทำการ mount sd card ด้วยการคลิก partition ที่สร้างใน Files
![](https://paper-attachments.dropbox.com/s_49F8656931A3E6AB0DFDD30FE17E245F04634FF25E3B671DA9A8CDA41D1035D2_1610960842672_Screenshot+from+2021-01-18+16-07-13.png)

## STEP 3.2 : นำไฟล์ที่เตรียมไว้ ลงใน sd_card
3. หลังจาก clone มาทำการแตกไฟล์ และนำไฟล์ test ที่เตรียมไว้มาทดสอบมาอยู่ในโฟลเดอร์เดียวกัน
    $ mkdir -p ~/Vitis-AI/ultra96v2_env_zcu104/{arch of dpu}_{core}_{dpu frequency}/model && cd ~/Vitis-AI/ultra96v2_env_zcu104/{arch of dpu}_{core}_{dpu frequency} # If you want to test another architech version on your work please make other directory like this {arch of dpu}_{core}_{dpu frequency}
    $ scp -r embedded@{server ip}:~/titan/Vitis-AI-Project/source/software/Vitis-AI/ultra96v2_env_zcu104/{arch of dpu}_{core}_{dpu frequency}/sd_card . 
    $ scp -r embedded@{server ip}:~/titan/Vitis-AI-Project/source/software/Vitis-AI/ultra96v2_env_zcu104/{arch of dpu}_{core}_{dpu frequency}/modelzoo/compiled_output/* ./model/. #Please Compile all AI model,that you want test before scp,That will save your time 
    $ git clone https://github.com/maxmamaximus/easyinitial_zcu104.git 
    $ cd easyinitial_zcu104
    $ cp -r ../model/compiled_output/* .
    $ cp interface ../sd_card/.


1. เสียบ SD card ที่เตรียมไว้ แล้วย้ายไฟล
    $ cd ../ && cp -r model ../easyinitial_zcu104
    $ cp -r sd_card/* /media/{user}/BOOT/.
    $ sudo cp root/* /media/{user}/rootfs/.
    $ sudo tar -zxvf /media/{user}/rootfs/rootfs.tar.gz
    # Setting Network
    $ sudo cp sd_card/interface /media/{user}/rootfs/etc/network/. # Don't forget to setting IP on your host machine before using 
5. นำ sd card ที่เตรียมเสียบเข้ากับบอร์ด zcu104 พร้อมทั้งนำสายแลนมาเสียบกับโน้ตบุ้ค
6. ทำการ secure shell เข้าไป
    $ ssh root@192.168.254.1
7. เปิดอีก terminal ทำการ secure copy ไฟล์ที่ clone มาใช้งาน
    $ scp -r {arch of dpu}_{core}_{dpu frequency}/easyinitial_zcu104 root@192.168.254.1:~/.
8. กลับมาที่บอร์ด แล้วเปิดไฟล์ทีโยนมา
    $ cd easyinitial_zcu104 
9. ผมได้ทำไฟล์ script ที่ติดตั้ง package เช่น dexplorer, vaitraceและการแก้ปัญหา PMIC แต่ละอย่างไว้หมดในไฟล์สคริปต์เดียวครับสามารถรันได้เลย 
    $ ./set_vitis_env.sh 
10. หลังจากติดตั้งเสร็จแล้วเราจะมาทดสอบการทำงานของ DPU กันครับ
----------
# STEP 4 : ทดสอบการทำงานของ DPU
1. ทดสอบ DPU spec
    $ dexplorer -w 
![](https://paper-attachments.dropbox.com/s_49F8656931A3E6AB0DFDD30FE17E245F04634FF25E3B671DA9A8CDA41D1035D2_1611306630909_dexplorer.png)

2. เอาโมเดล resnet50 ที่เทรนมาไปใช้งาน
    $ cp ./model/cf_resnet50_imagenet_224_224_7.7G_1.2/*.elf /usr/share/vitis_ai_library/models/resnet50/.  
    $ rm -rf /usr/share/vitis_ai_library/models/resnet50/resnet50.elf && mv /usr/share/vitis_ai_library/models/resnet50/dpu*.elf /usr/share/vitis_ai_library/models/resnet50/resnet50.elf 
    #If you want to try another model, please change old model to your compiled model like this 
    #$ cp ./model/{model_zoo_name}/*.elf /usr/share/vitis_ai_library/models/{model_name}/. 
![](https://paper-attachments.dropbox.com/s_2F947DCF497139D9EB76B8016A5AE9A8E1C92D04D7BA4CED17D5FF60F97258C8_1604462653723_Screenshot+from+2020-11-04+11-03-54.png)

2. แตกไฟล์ vitis-ai.tar.gz
    $ tar -zxvf vitis-ai.tar.gz
3. เข้าไปทดสอบโมเดลที่เตรียมไว้
    $ cd /overview/sample/resnet50
    $ ./test_jpeg_resnet50 resnet50 sample_jpeg.jpg


![](https://paper-attachments.dropbox.com/s_49F8656931A3E6AB0DFDD30FE17E245F04634FF25E3B671DA9A8CDA41D1035D2_1610093015389_file.jpeg)



----------
# SPEP 5 : Test Bench AI module 

Reference : [Vitis-AI/Vitis-AI-Profiler at v1.2.1 · Xilinx/Vitis-AI (github.com)](https://github.com/Xilinx/Vitis-AI/tree/v1.2.1/Vitis-AI-Profiler)

1. เตรียมโมเดล yolov3_adas_pruned_0_9 
    $ cp ./model/cf_resnet50_imagenet_224_224_7.7G_1.2/*.elf /usr/share/vitis_ai_library/models/resnet50/.  
    $ rm -rf /usr/share/vitis_ai_library/models/yolov3_adas/yolov3_adas_pruned_0_9.elf && mv /usr/share/vitis_ai_library/models/yolov3_adas/dpu*.elf /usr/share/vitis_ai_library/models/yolov3_adas/yolov3_adas_pruned_0_9.elf 
2. ทำการเก็บประสิทธิภาพขณะทำงานด้วย Vaitrace เป็น tools นึงใน vart package ใช้ในการสร้าง Profile การทำงานงานของบอร์ดขณะใช้งาน AI application
    $ cd ~/easyinitial_zcu104/Vitis-AI/vitis-ai-library/overview/samples/yolov3
    $ vaitrace -t 10 -o ~/trace_yolov3.xat ./test_performance_yolov3 yolov3_adas_pruned_0_9 ./test_performance_yolov3.list
    # -t is time used for benchmark
    # -o is output file
    # argument 0 : ./test_performance_yolov3 AI application used on testing
    # argument 1 : yolov3_adas_pruned_0_9 model used on testing 
    # argument 2 : ./test_performance_yolov3.list is list image on testing 
    
    #optional
    #if you want to check RAM used while running please open other terminal on ssh board.
    $ watch -n 1 free

 3. กลับมาที่ terminal ของ host machine นำไฟล์กลับไปที่เครื่อง host โดยการ scp

    $ scp -r root@192.168.254.1:~/trace_yolov3.xat ~/. 

 4. ทำการติดตั้ง environment และเปิด python server 

    $ cd /path-to-vitis-ai/Vitis-AI-Profiler
    $ pip3 install --user flask
    $ python3 vaiProfiler.py 
    # After this get ip and open local host on web-browser pick file .xat, let's ploting benchmark graph
![](https://paper-attachments.dropbox.com/s_49F8656931A3E6AB0DFDD30FE17E245F04634FF25E3B671DA9A8CDA41D1035D2_1611309486866_image.png)

----------

สรุปงาน การใช้งาน dpu บนบอร์ด zcu104 นั้นมีทรัพยากรเพียงพอถ้านำไปปรับบน Ultra96 ที่แก้ปัญหา PMIC แล้วอาจใช้งานได้ แต่จากที่ผมเคย build dpu ultra96 board นั้นทรัพยากรเพียง 1 core ก็ไม่พอแล้วครับ 



