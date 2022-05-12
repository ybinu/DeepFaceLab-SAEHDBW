# DeepFaceLab - New Grayscale SAEHDBW Model and Code Review / Documentation of the source files

## Notes, experience, tools, deepfakes

~ 22.4.2022 -- Minor iterface changes (more keys for save, save preview periods and auto saving;later: possible forcing generation of new previews instead of keeping the same for the whole training etc.; reviewing the code

~ 25.4.2022? --> Started working on SAEHDBW - Grayscale deepfake model; research, experiments, modifications of the channel dimensions, studying the NN model.

## Goals of the project:
* Allow training of more "respectable" resolution models even on 2 GB GPUs, GeForce 750 Ti in particular, and on integrated GPUs
* Achieve several times? higher performance: either smaller models, higher resolution and/or more detailed models, although grayscale. 
* Study the code, if possible modify the architectures and optimize more: simplify/reduce the depth of some networks to check if they would achieve similar quality due to the single channel with improved performance. 

* ~ 27.4.2022 --> SAEHDBW - SUCCESS!

First correctly training version (last error fixed masks getting bug after untested change, numpy).
Initial mode: training from color input which is converted to grayscale during reading.
Now the model can train on 224x224 and 192x192 images on a 2 GB Geforce 750 Ti. (The quality etc. is to be checked at various AE dims, encoder dims, decoder dims.)

liae-ud-r192-ae-96-48-48-12-bw_SAEHDBW  x f, ~900ms (R192...-ae-64-48-48 - almost the same it time ~ 860 ms)

R224-AE64-48-48-12-BW_SAEHDBW x f liae-ud ~ 1500-1600ms

Training on converted to grayscale pretrain faceset, resized to 384x384 (from 768x768). Checking how much details would be captured with different dimensions.

2.5.2022:

DF-UDT-256-96-32-32-16_SAEHDBW --> batch 4: ~ 1200 ms, (~ 1150 ms slightly more overclock); batch 5 ~ 1500 ms (OOM errors occasionaly)

Some model sizes on disk and batch size (for 750 Ti/2 GB)

```
Model Sizes: MB

LIAE-UD

liae-ud-r192-ae-96-48-48-12-bw_SAEHDBW -- 362 MB  (12 is == 16 mask dim)

R192-AE80-48-48-16-LIAE-UD-SAEHD-BW-PRETR_SAEHDBW-- 315 M
R192-AE64-48-48-16-LIAE-UD-SAEHDBW-PRETR_SAEHDBW  -- 269 M

liae-ud-r96-64-64-22_SAEHDBW -- 313 M
R224-AE64-48-48-12-BW_SAEHDBW -- 297 M  (12 is == 16)

liae-ud-r96-24-24-12_SAEHDBW -- 45.6 M
liae-ud-r96-32-32-12-bw_SAEHDBW -- 96 M

LIAE-UDT-R128-96-32-32-16_SAEHDBW -- 209 M  B: 4,6,8 (B=8: it= 444-463 ms (530, Lower power mode) --> ~4K@4, 13K@6 --> 8), 4.5.2022 --> train at f (also do on mf)

LIAE-UDT-192-128-32-32-16-SAEHDBW_SAEHDBW_summary -- 270 MB
LIAE_UDT-192-96-32-48-16-SAEHDBW_SAEHDBW_summary -- 346 MB ==> would it be beneficial if having a lower dim. encoder than decoder (yet more parameters overall and thus more detail?)

LIAE-UDT 192-96-32-48-16 vs LIAE-UD 192-96-48-48-16 ?


"G:\SAEHDBW\liae-udt-192-96-32-32-16-SAEHDBW_SAEHDBW... - 234 МБ

"G:\SAEHDBW\liae-ud-192-128-32-32-16_SAEHDBW_src_dst_opt.npy" - 273 МБ

*** DF-UD ***

dfud-r96-32-32-12-bw_SAEHDBW_summary.txt -- 104 M
DF-UDT-256-96-32-32-16_SAEHDBW -- 281 M B: 4, 5 (OOM in minutes sometimes)
DF-UDT-R96-64-24-24-16-SAEHDBW_SAEHDBW -- 50 MB , train @mf 

"G:\SAEHDBW\df-udt-192-96-32-32-16_SAEHDBW - 285 МБ

"G:\SAEHDBW\ df-udt-192-128-48-32-16_SAEHDBW - 345 МБ

df-ud-192-128-48-48-16_SAEHDBW_summary - 345 МБ

(Check the quality of df-ud and df-udt with the same number of params - if there's enough patience to train them. Does 48-32 is good enough, varying number of channels/dimensions for the encoder and decoder? Encoder > decoder ... Also: 128/32/32? Mapping the default 256/64/64 for color 128 pix)

```

Trying 't', searching for higher sharpness; various settings tried.

* Only 1.45 GB available. Connecting the monitor to the integrated GPU, but OS still reserves that amount and sometimes even with just 77% usage when connecting a monitor to the integrated GPU, after trying to create a bigger batch the model doesn't start training. (Windows 10 issue.)

* 10-5-2022 - Debugged the CUDA build so now I can use it. (I used the DirectX12 build so far, because the CUDA-one hanged with no output). The solution provided 33% speed-up! https://github.com/iperov/DeepFaceLab/issues/5515 "device_lib.list_local_devices() doesn't return in the CUDA build up to 2080 #5515"
* 
* 11.5.2022 - After a series of GPU-related crashes when trying to run big models at the edge in the CUDA build, with sizes which previously were training in the DirectX12 version, it seems that the DX12 version, i.e. tensorflow-directml uses less memory than CUDA. It is possible to train: DF-UDT-256-96-32-32-16_SAEHDBW --> as recorded recently: batch 4: ~ 1200 ms, (~ 1150 ms slightly more overclock); batch 5 ~ 1500 ms (OOM errors occasionaly).

## Sample pretraining and training experiments

* SAEHDBW df-udt-mf-R192-128-48-32-16, batch 6 pretraining on the custom subset of the built-in faceset.pak, reextracted* at 384x384 grayscale and with removed many images, lastly about 14634. It still has a few "bad" samples with babies and hand in the mouth, some musician with an instrument etc, hair covering some of the eye etc. Portraits with glasses are kept, except extremely strange ones. Microphones, hands and other objects crossing the face are removed, expect a few and when it is slightly touching; etc.

* 12.5.2022 Note: They had better be resized instead of reextraceted, using a modified DFL script for resizing, but I haven't reached that part of the code then.

* Still training:

**348К-349K**

![image](https://user-images.githubusercontent.com/23367640/168179786-1629ffc4-27ec-4434-b992-f21af12fb870.png)

![image](https://user-images.githubusercontent.com/23367640/168178090-4a053abf-6242-4dd1-baf5-2c5862d4f9c3.png)
![image](https://user-images.githubusercontent.com/23367640/168178227-183cc653-2fe5-4ba1-a927-ebc49e18c8d5.png)
![image](https://user-images.githubusercontent.com/23367640/168178349-d42376c5-0396-4df7-93b9-3937fb303790.png)
![image](https://user-images.githubusercontent.com/23367640/168178496-b54f35a8-f0a8-4dcf-bd48-749be7dfce02.png)

Some pale teeth-splits. possibly mask- and face-type-related (mf):

![image](https://user-images.githubusercontent.com/23367640/168179163-97bbf9f5-df0e-406d-99f5-5a41b5c68788.png)

Still there are teeth which are not split, maybe due to the rare combination: only close to the lower lip. Usually the teeth are only up, below the upper lip.

![image](https://user-images.githubusercontent.com/23367640/168178667-35f68825-783e-4906-bba8-bb98dd80807d.png)



**340K**

![image](https://user-images.githubusercontent.com/23367640/168166318-8cbe33ba-a401-487e-9074-40f764d3e738.png)
![image](https://user-images.githubusercontent.com/23367640/168178152-7611fbd0-23af-453d-9faa-19e4b058bb0c.png)

**336K**

![image](https://user-images.githubusercontent.com/23367640/168158108-64121437-ca19-4cb6-882f-bbad24989ddf.png)
![image](https://user-images.githubusercontent.com/23367640/168158552-5901ab3c-c186-49e1-bcb7-185feb0f39ff.png)
![image](https://user-images.githubusercontent.com/23367640/168158335-bdea3786-d9a2-457d-9c6b-5bf0ca91be12.png)


**333K**

![image](https://user-images.githubusercontent.com/23367640/168144862-2946cde9-d05a-4d07-b8fb-e5149631ed05.png)
![image](https://user-images.githubusercontent.com/23367640/168145312-0ab62709-9df5-4195-a33c-cec9f7e1d92e.png)


**304К**

![image](https://user-images.githubusercontent.com/23367640/168078625-5782ea33-f880-4f06-8e3f-e7c7433d095e.png)
![image](https://user-images.githubusercontent.com/23367640/168078934-f8b9e1d0-f9fb-4e42-8e12-73f9289c0ce5.png)
![image](https://user-images.githubusercontent.com/23367640/168079007-1df3f993-97ab-4788-9c73-083f880e3e31.png)
![image](https://user-images.githubusercontent.com/23367640/168079062-72c232bb-9067-401a-a06b-6f030e3a2d52.png)
![image](https://user-images.githubusercontent.com/23367640/168079124-ac7ce422-7060-424f-9c97-34c37ec085bb.png)



**302K**

![image](https://user-images.githubusercontent.com/23367640/168074945-5b931dfd-1409-4d57-8328-a36c1247de3c.png)
![image](https://user-images.githubusercontent.com/23367640/168074986-c2ccdcb1-9d59-491a-82b9-8fe081a83380.png)
![image](https://user-images.githubusercontent.com/23367640/168075071-c627c15d-b52f-4b66-a903-f89ae07b616d.png)
![image](https://user-images.githubusercontent.com/23367640/168075120-0851ce72-f65a-45e8-9d4b-11972651b2ae.png)



**296K-297K** 12.5.2022

![image](https://user-images.githubusercontent.com/23367640/168064008-4a8a9d4b-012d-49cc-84dc-df33b7fedd36.png)

Still some pale teeth's splits - probably due to the rare pose (widely open mouth).

As expected, this resolution model can't capture the facial relief of older people's portraits, but this is still the pretrain: will it represent at least some relief for two faces at this resolution? It did manage for the Biden model at liae-ud-96-96-32-32.

[To do: Continue the training also of df-ud-192-128-48-48-16, df-ud 256-96-32-32 (DX12 only) and compare, and train also higher AED(M) dimension models for lower resolution, to see whether lower dimension models would not capture similar detail, due to the single-channel input.]


![image](https://user-images.githubusercontent.com/23367640/168064178-ba1445fc-eb8c-43a1-9110-1334d3a84c6d.png)

Frontal teeth sometimes are very good, but still a bit low contrast

![image](https://user-images.githubusercontent.com/23367640/168064934-fe61b7f2-91a1-40c4-aebf-95511e05c148.png)



**250K** ~ 1.7466/1.7354

![image](https://user-images.githubusercontent.com/23367640/167980250-0efd7387-8bc5-4a14-b134-05cc91522592.png)
![image](https://user-images.githubusercontent.com/23367640/167980321-c415cbd0-cc50-4693-afb2-eac0f444d81f.png)
![image](https://user-images.githubusercontent.com/23367640/167980479-6816ab9b-bb8b-4d9d-84ab-3108f5783659.png)


**240K-241K** - well developed teeth in many samples

![image](https://user-images.githubusercontent.com/23367640/167969467-b43934c5-045f-4621-866d-a335c4f1de41.png)
![image](https://user-images.githubusercontent.com/23367640/167969537-fad5bede-c8ee-49e0-a5c4-6d075227b68c.png)
![image](https://user-images.githubusercontent.com/23367640/167969663-d409e124-a2a5-4c5e-9f74-944d10145b5d.png)
![image](https://user-images.githubusercontent.com/23367640/167970000-0b062d31-0a65-4c62-983a-4d6eaf238366.png)
![image](https://user-images.githubusercontent.com/23367640/167970149-07dea55e-ed89-4322-a944-beb7351c68e0.png)
![image](https://user-images.githubusercontent.com/23367640/167970790-1c8c705b-eb55-4f6f-9778-22f59af92e63.png)

Glasses:

![image](https://user-images.githubusercontent.com/23367640/167969723-e9c29513-b847-431e-82c6-9b028dcc3893.png)
![image](https://user-images.githubusercontent.com/23367640/167969772-b5247f32-55c8-4bbc-8341-db50cc241abc.png)
![image](https://user-images.githubusercontent.com/23367640/167969819-afb3c2c5-98a7-4e44-ab30-118bd8279e43.png)
![image](https://user-images.githubusercontent.com/23367640/167969850-18717915-e9b1-42d6-baeb-fd68d3970c8f.png)

Detailed eyes with irises. Even highlights sometimes.

![image](https://user-images.githubusercontent.com/23367640/167970114-93b0f9c5-a669-4884-a09a-3f3a86c48917.png)
![image](https://user-images.githubusercontent.com/23367640/167970242-f2f9f981-99e9-4e67-a8c3-4a1ee2b77edd.png)
![image](https://user-images.githubusercontent.com/23367640/167970274-5e04667d-5d14-4a71-a5a4-f21f19502f43.png)
![image](https://user-images.githubusercontent.com/23367640/167970418-fc9d6792-7921-4c8f-a88f-e8fdc991b21a.png)

**225K-231K**

![image](https://user-images.githubusercontent.com/23367640/167972416-87c81808-c5bf-40f9-96f1-b72902b3add7.png)
![image](https://user-images.githubusercontent.com/23367640/167972460-e2cfbc75-3796-497f-a3e7-35d99735c949.png)
![image](https://user-images.githubusercontent.com/23367640/167972500-27d63e9d-502e-4836-8cdb-9558aea0fe11.png)
![image](https://user-images.githubusercontent.com/23367640/167972514-087f6f63-11fa-4440-9ead-02f22c125297.png)
![image](https://user-images.githubusercontent.com/23367640/167972531-f81ca0d0-1fee-4e09-9a59-dbd477744d3f.png)
![image](https://user-images.githubusercontent.com/23367640/167972549-9439d97a-1b00-44f3-b51d-014e6cfd032e.png)
![image](https://user-images.githubusercontent.com/23367640/167972597-c0a6da3c-ed26-4084-b356-ef9ab978340a.png)
![image](https://user-images.githubusercontent.com/23367640/167972606-f328bbb2-2c03-4ffc-97d0-613211dce386.png)
![image](https://user-images.githubusercontent.com/23367640/167972654-4e772d24-1783-492b-b6ac-ae18262b4fa8.png)
![image](https://user-images.githubusercontent.com/23367640/167972703-5ea47bdc-dbea-4493-b0bd-793ddb4106a6.png)
![image](https://user-images.githubusercontent.com/23367640/167972711-f4dff91f-9b66-4b92-b4a4-f5480b7e1a91.png)
![image](https://user-images.githubusercontent.com/23367640/167972726-a1d097b4-d9cd-4578-8a80-0f0f87c44c4c.png)
![image](https://user-images.githubusercontent.com/23367640/167972741-9fa06d4f-4b82-4023-9245-719d7eccb26e.png)
![image](https://user-images.githubusercontent.com/23367640/167972752-70eb5f42-614d-4700-8b63-4ccfaa59a651.png)
![image](https://user-images.githubusercontent.com/23367640/167972789-4e948326-7ca5-4ec9-81c7-880e2a5a7d1f.png)
![image](https://user-images.githubusercontent.com/23367640/167972813-049205db-2d0f-4fdb-8e64-29c81a815c93.png)
![image](https://user-images.githubusercontent.com/23367640/167972834-037e20ee-ab43-42a8-bf36-b2ad37d191e1.png)


**214K**

![image](https://user-images.githubusercontent.com/23367640/167972377-b4454f86-f74d-4eec-99c1-b082e80a6630.png)


**205K-206K**

Some teeths are forming, other smiles are too bright. Some glasses.

![image](https://user-images.githubusercontent.com/23367640/167972088-6d13b780-7ca8-4796-ba33-e40afe1cf8ae.png)
![image](https://user-images.githubusercontent.com/23367640/167972107-4adf0a8d-eaee-47f3-963f-9907ca8751f9.png)
![image](https://user-images.githubusercontent.com/23367640/167972136-27c6a451-d764-46fc-9571-d77d43bbee39.png)
![image](https://user-images.githubusercontent.com/23367640/167972146-413dd44f-7e35-43d4-a483-670aa38e7f42.png)
![image](https://user-images.githubusercontent.com/23367640/167972160-0894cc54-e248-401f-9815-b8a7d367b2ea.png)
![image](https://user-images.githubusercontent.com/23367640/167972181-9153fd34-c797-4865-9ce8-e94c349fd3a3.png)
![image](https://user-images.githubusercontent.com/23367640/167972198-694abe09-0a53-4415-9703-c08e37a36bd6.png)
![image](https://user-images.githubusercontent.com/23367640/167972230-b209484c-ba34-4e8d-a03a-daff7fb436d4.png)
![image](https://user-images.githubusercontent.com/23367640/167972273-bfe84412-6711-4bf7-a0d6-256ac267b576.png)
![image](https://user-images.githubusercontent.com/23367640/167972286-41e70786-e63d-4207-93b5-37b8f21a4a0e.png)


**183K-185K**

![image](https://user-images.githubusercontent.com/23367640/167971813-bcccf5dc-ee46-4a12-beb4-41585883ec8c.png)
![image](https://user-images.githubusercontent.com/23367640/167971897-a93bb7c3-9ecc-4ae6-9cf3-3da43792805d.png)
![image](https://user-images.githubusercontent.com/23367640/167971921-52ab80af-e517-4ce7-8e9d-9a4da8be0057.png)
![image](https://user-images.githubusercontent.com/23367640/167971942-767ca749-dbb1-43b3-a65d-ad9b51761e32.png)
![image](https://user-images.githubusercontent.com/23367640/167971958-a5db0d44-32e8-46da-ada1-7a9634ee5453.png)
![image](https://user-images.githubusercontent.com/23367640/167971979-2a8035fe-519d-40a4-9650-87af6df690c6.png)
![image](https://user-images.githubusercontent.com/23367640/167971999-5b1aff52-48c2-4d99-8204-2f933ec19f66.png)
![image](https://user-images.githubusercontent.com/23367640/167972037-39936eea-44d9-4683-8056-68800363d2be.png)


**171K**

![image](https://user-images.githubusercontent.com/23367640/167971708-40e75ca4-4d87-470f-8033-f16803b42554.png)
![image](https://user-images.githubusercontent.com/23367640/167971745-4772b688-d32c-4772-ad85-a55072a94721.png)
![image](https://user-images.githubusercontent.com/23367640/167971787-e020dd08-a83c-4b3e-bebe-aeab6e7cbdd6.png)


**161K**

![image](https://user-images.githubusercontent.com/23367640/167971615-ddbe1115-02e1-44d4-8b2d-af9941e702f3.png)

![image](https://user-images.githubusercontent.com/23367640/167971638-a3dc6ea4-9db4-4d47-9f31-7833b77dbc34.png)
![image](https://user-images.githubusercontent.com/23367640/167971662-4222a784-f537-4a9c-9629-eada2b8d9b4b.png)

**151K**

The middle split of the teeth is visible on some of the samples, but is still pale:

![image](https://user-images.githubusercontent.com/23367640/167971544-69bde746-fe60-49cc-8ba5-3af0962c9779.png)


**116K**

![image](https://user-images.githubusercontent.com/23367640/167688496-bd17aeb8-12cb-433f-9503-13e98ced5a8a.png)
![image](https://user-images.githubusercontent.com/23367640/167688596-488dbfd3-55c4-47bc-9771-385aa93a57ff.png)
![image](https://user-images.githubusercontent.com/23367640/167688617-41a41380-1719-48e4-bceb-834291e0e700.png)
![image](https://user-images.githubusercontent.com/23367640/167688640-a3399de1-f12b-4247-a877-2bf79dcb019d.png)
...

**87К**

![image](https://user-images.githubusercontent.com/23367640/167518631-674dbefd-6fcf-42c4-aef1-d13e1e4a62dc.png)

**61К-62К**

![image](https://user-images.githubusercontent.com/23367640/167448002-411f0a06-11fb-49ed-ae25-1b63eba3adc8.png)
![image](https://user-images.githubusercontent.com/23367640/167447405-3b089aca-6a19-4ab6-86d4-12fb7d8d1ad5.png)
![image](https://user-images.githubusercontent.com/23367640/167447472-e1a51ab4-f759-44d9-a632-96cacbf6ff61.png)

```
...

**Sample Loss history**

[21:54:11][#027222][1050ms][2.6504][2.4689]
... On saves:
[19:28:09][#063712][1062ms][2.0886][2.0911]
[19:42:29][#064509][1046ms][2.0791][2.0771]
[19:57:30][#065349][1049ms][2.0869][2.0834]
[20:12:30][#066184][1066ms][2.0749][2.0683]
[20:27:30][#067012][1048ms][2.0866][2.0733]
[20:42:30][#067846][1057ms][2.0501][2.0615]
[21:37:49][#068808][1039ms][2.0612][2.0680]
[21:52:11][#069586][1057ms][2.0786][2.0695]
[22:07:10][#070426][1058ms][2.0583][2.0569]
[22:22:11][#071271][1040ms][2.0516][2.0413]
[22:37:11][#072114][1044ms][2.0497][2.0640]
[22:52:10][#072962][1047ms][2.0541][2.0489]
[23:07:11][#073811][1054ms][2.0484][2.0384]
[23:22:10][#074646][1060ms][2.0365][2.0487]
[23:37:11][#075488][1063ms][2.0469][2.0329]
[23:52:10][#076327][1049ms][2.0374][2.0298]
[00:07:11][#077171][1051ms][2.0355][2.0301]
[00:22:10][#078015][1049ms][2.0254][2.0313]
[00:37:11][#078860][1061ms][2.0211][2.0269]
[00:52:10][#079703][1083ms][2.0165][2.0280]
[01:07:11][#080547][1050ms][2.0227][2.0232]
[01:22:12][#081363][1131ms][2.0262][2.0016]
[01:37:11][#082116][1201ms][2.0134][1.9998]
[01:52:12][#082879][1108ms][2.0180][2.0309]
[02:07:11][#083638][1426ms][2.0129][2.0045]
[02:22:11][#084418][1150ms][2.0109][2.0119]
[02:37:11][#085188][1313ms][2.0112][2.0040]
[02:52:11][#085942][1186ms][1.9865][1.9941]
[03:07:11][#086694][1209ms][2.0134][1.9994]
[03:10:08][#086830][1184ms][1.9840][1.8104]

...

[02:08:03][#232699][0740ms][1.7640][1.7560]
[02:10:46][#232937][0880ms][1.7459][1.7616]
[02:25:44][#234161][0688ms][1.7655][1.7586]
[02:40:46][#235443][0881ms][1.7547][1.7553]
[02:55:46][#236751][0843ms][1.7550][1.7543]
[03:10:47][#238023][0801ms][1.7599][1.7516]
[03:25:47][#239291][1006ms][1.7487][1.7582]
[03:40:46][#240554][0723ms][1.7608][1.7471]
[03:55:46][#241798][1073ms][1.7505][1.7526]
[04:10:46][#243044][0856ms][1.7485][1.7529]
[04:25:46][#244292][0808ms][1.7465][1.7463]
[04:40:46][#245586][0715ms][1.7554][1.7491]
[04:55:45][#246873][0819ms][1.7387][1.7498]
[05:10:46][#248123][0751ms][1.7527][1.7564]
[05:25:43][#249415][0687ms][1.7466][1.7354]
[05:40:46][#250692][1124ms][1.7527][1.7416]
[05:55:43][#251974][0657ms][1.7453][1.7392]
[06:10:45][#253317][0653ms][1.7467][1.7420]
[06:25:46][#254644][0921ms][1.7430][1.7391]
[06:40:46][#255968][0939ms][1.7394][1.7379]
[06:55:46][#257293][0954ms][1.7462][1.7379]
[07:10:46][#258618][0912ms][1.7357][1.7406]
[07:25:46][#259944][0936ms][1.7413][1.7319]
[07:40:46][#261271][0829ms][1.7319][1.7341]
[07:55:46][#262594][0834ms][1.7352][1.7371]
[08:10:46][#263920][0801ms][1.7333][1.7366]
[08:25:45][#265246][0844ms][1.7353][1.7362]
[08:40:47][#266574][1460ms][1.7350][1.7339]
[08:55:46][#267900][0852ms][1.7296][1.7278]
[09:10:46][#269224][0824ms][1.7271][1.7309]
[09:25:46][#270549][0817ms][1.7312][1.7386]
[09:40:46][#271874][0803ms][1.7325][1.7182]
[09:55:46][#273199][0792ms][1.7217][1.7200]
[10:10:46][#274522][0832ms][1.7365][1.7356]
[10:25:45][#275847][0783ms][1.7260][1.7240]
[10:40:45][#277170][0857ms][1.7225][1.7197]
[10:55:46][#278464][0926ms][1.7196][1.7180]
[11:10:46][#279785][0906ms][1.7360][1.7305]
[11:25:46][#281105][0879ms][1.7163][1.7180]
[11:40:45][#282425][0927ms][1.7250][1.7221]
[11:55:46][#283746][0935ms][1.7198][1.7171]
[12:10:46][#285066][1022ms][1.7255][1.7158]
[12:25:46][#286386][0903ms][1.7185][1.7149]
[12:40:45][#287705][0785ms][1.7164][1.7208]
[12:55:46][#289025][0906ms][1.7185][1.7129]
[13:10:46][#290344][0968ms][1.7169][1.7155]
[13:25:45][#291664][0890ms][1.7160][1.7096]
[13:40:46][#292984][0959ms][1.7147][1.7209]
[13:55:46][#294303][0901ms][1.7225][1.7085]
[14:10:46][#295604][0967ms][1.7122][1.7127]
[14:25:48][#296823][0740ms][1.7018][1.7079]
[14:40:47][#298082][0656ms][1.7120][1.7077]
[14:55:47][#299316][1457ms][1.7109][1.7024]
[15:10:44][#300559][0633ms][1.7162][1.7097]
[15:25:46][#301872][0862ms][1.7068][1.7103]
[15:40:47][#303137][0827ms][1.7038][1.7041]
[15:55:46][#304391][0680ms][1.7045][1.7040]
[16:10:46][#305604][0974ms][1.7048][1.7133]
[16:25:44][#306920][0635ms][1.7199][1.6983]
[16:40:47][#308171][0967ms][1.6995][1.7069]
[16:55:47][#309391][1505ms][1.7019][1.6912]
[17:10:46][#310656][0970ms][1.7059][1.7027]
[17:25:46][#311970][0883ms][1.7061][1.7019]
[17:40:46][#313283][1031ms][1.7025][1.7025]
[17:55:46][#314595][0951ms][1.6950][1.7091]
[18:10:46][#315910][1097ms][1.7095][1.6913]
[18:25:47][#317224][0851ms][1.7068][1.6967]
[18:40:47][#318536][1398ms][1.6991][1.6951]
[18:55:46][#319849][1091ms][1.7034][1.7044]
[19:10:46][#321162][1076ms][1.6971][1.6950]
[19:25:47][#322477][1117ms][1.6936][1.6939]
[19:40:46][#323790][1020ms][1.6883][1.6912]
[19:55:46][#325102][1071ms][1.6936][1.6966]
[20:10:46][#326307][0963ms][1.7064][1.6882]
[20:25:45][#327604][1086ms][1.6967][1.6927]
[20:40:45][#328881][0961ms][1.6863][1.6938]
[20:55:46][#330131][1049ms][1.6894][1.6914]
[21:10:45][#331378][1026ms][1.6899][1.6880]
[21:25:46][#332634][1565ms][1.6893][1.6878]

```

* SAEHDBW liae-ud-r96-32-32-16; no pretraining; mouth and eye priority

Iterations: about 220-230 ms for batch size 8 and about 320-330 for 12 after 370K, training from color images. This model is still progressing. The faceset is currently about 400 images for Biden and 2200 images for the other person (K.P.), where the Biden's are of a higher quality and sharper. Almost all of K's images are from videos and low resolution. Initially training only on about 200 stills of Biden, then about 200 frames from a clip of a lower quality. The model has small dimensions so very high quality can't be captured anyway.

![image](https://user-images.githubusercontent.com/23367640/166139358-17555ce6-06b0-4089-98be-41a2d596e34f.png)

* Modified faceset - reduced to about 14600 images, removed many which I didn't "like", having occlusions - microphones, hands etc., grayscale 384x384



* SAEHDBW liae-udt-r128-96-32-32-16 Pretraining; mouth and eye priority

![image](https://user-images.githubusercontent.com/23367640/167268956-ed171f28-8132-4762-ad9b-daf48c11ee17.png)
![image](https://user-images.githubusercontent.com/23367640/167268964-7154019d-46a4-4629-bd34-de72f65b4466.png)
![image](https://user-images.githubusercontent.com/23367640/167268973-dc30dad3-d080-4c3f-b21b-fbb6fd8793b1.png)
![image](https://user-images.githubusercontent.com/23367640/167268983-7240d846-ec19-4aa9-a125-d0a35aabf9d0.png)
![image](https://user-images.githubusercontent.com/23367640/167268986-50996c10-6542-46fa-b712-a9a1b4464625.png)

...


Continuing working on the project.

Training with grayscale input (8-bit jpg, png) significant improvement (twice) for the pre-training dataset color images.
Modify Extract to extract to grayscale.
Unpacking the pretrain faceset, 768x768 color. Extract to 384x384 BW.

19.4.2022

```
preview_period = 334
preview_period_colab = 200

C:\DFL\DeepFaceLab_DirectX12\_internal\DeepFaceLab\models\Model_SAEHD\Model.py
  #override
    def should_save_preview_history(self):
       return (not io.is_colab() and self.iter % preview_period == 0) or (io.is_colab() and self.iter % preview_period_colab == 0)
    #    return (not io.is_colab() and self.iter % ( 10*(max(1,self.resolution // 64)) ) == 0) or \
    #           (io.is_colab() and self.iter % 100 == 0)

Also in ModelBase.py

Similar function, ... but it's overriden in the separate models.
```


VS: copy with line numbers 


17.4.2022

Reduce noise, training at home/PC in the bedroom:
Намаляване на шума при обучение вкъщи на компютър в спалнята (през нощта):

* Win 10, Power Options/Опции на захранването: Икономичен режим, до 40-50% на ЦПУ - спира да бучи циклично (Core i5 6500 3.3 GHz).

DFL Colab - от оригиналния iperov ... e... drive - edge --> 


9.4.2022

* https://mrdeepfakes.com/forums/thread-guide-deepfacelab-2-0-guide#
* https://github.com/iperov/DeepFaceLive
* https://github.com/iperov/DeepFaceLab

Try also Live

* If downloading the github repo and running the main.py as with Colab, it will run only on CPU. It must be built.
* If using the CPU version and newer OpenCV than 4.1, edit:
``` 
C:\Deepface\core\imagelib\warp.py

Locate:

def gen_warp_params ...

 random_transform_mat = cv2.getRotationMatrix2D((int(w // 2), int(w // 2)), rotation, scale) 
  random_transform_mat[:, 2] += (tx*w, ty*w)

#### (int(w // 2), int(w // 2)) #####
```

* Download the Builds and run/edit the .bat files
 
* After install, find trainer.py 
Change trainer thread: 

```
 while True:
        try:
            start_time = time.time()

            save_interval_min = 241 # 25  -- or whatever

```

To whatever you like.

* SAEHD 192

```
Running trainer.

Choose one of saved models, or enter a name to create a new model.
[r] : rename
[d] : delete

[0] : sae-192 - latest
[1] : ksaehd
 : 0
0
Loading sae-192_SAEHD model...

Choose one or several GPU idxs (separated by comma).

[CPU] : CPU
  [0] : NVIDIA GeForce GTX 750 Ti
  [1] : Intel(R) HD Graphics 530

[1] Which GPU indexes to choose? : 0
0

Press enter in 2 seconds to override model settings.
[0] Autobackup every N hour ( 0..24 ?:help ) :
0
[y] Write preview history ( y/n ?:help ) :
y
[n] Choose image for the preview history ( y/n ) : y
[3001] Target iteration : 10001
10001
[n] Flip SRC faces randomly ( y/n ?:help ) :
n
[y] Flip DST faces randomly ( y/n ?:help ) :
y
[4] Batch_size ( ?:help ) :
4
[y] Masked training ( y/n ?:help ) :
y
[y] Eyes and mouth priority ( y/n ?:help ) :
y
[n] Uniform yaw distribution of samples ( y/n ?:help ) :
n
[n] Blur out mask ( y/n ?:help ) :
n
[n] Place models and optimizer on GPU ( y/n ?:help ) :
n
[n] Use AdaBelief optimizer? ( y/n ?:help ) :
n
[n] Use learning rate dropout ( n/y/cpu ?:help ) :
n
[y] Enable random warp of samples ( y/n ?:help ) :
y
[0.0] Random hue/saturation/light intensity ( 0.0 .. 0.3 ?:help ) :
0.0
[0.0] GAN power ( 0.0 .. 5.0 ?:help ) :
0.0
[0.0] Face style power ( 0.0..100.0 ?:help ) :
0.0
[0.0] Background style power ( 0.0..100.0 ?:help ) :
0.0
[rct] Color transfer for src faceset ( none/rct/lct/mkl/idt/sot ?:help ) :
rct
[n] Enable gradient clipping ( y/n ?:help ) :
n
[y] Enable pretraining mode ( y/n ?:help ) :
y
Initializing models: 100%|###############################################################| 5/5 [00:01<00:00,  2.56it/s]
Loaded 15843 packed faces from C:\DFL\DeepFaceLab_DirectX12\_internal\pretrain_faces
Sort by yaw: 100%|##################################################################| 128/128 [00:00<00:00, 239.71it/s]
Sort by yaw: 100%|##################################################################| 128/128 [00:00<00:00, 238.36it/s]
Choose image for the preview history. [p] - next. [space] - switch preview type. [enter] - confirm.
=================== Model Summary ====================
==                                                  ==
==            Model name: sae-192_SAEHD             ==
==                                                  ==
==     Current iteration: 3001                      ==
==                                                  ==
==----------------- Model Options ------------------==
==                                                  ==
==            resolution: 192                       ==
==             face_type: wf                        ==
==     models_opt_on_gpu: False                     ==
==                 archi: liae-ud                   ==
==               ae_dims: 128                       ==
==                e_dims: 96                        ==
==                d_dims: 64                        ==
==           d_mask_dims: 16                        ==
==       masked_training: True                      ==
==       eyes_mouth_prio: True                      ==
==           uniform_yaw: True                      ==
==         blur_out_mask: False                     ==
==             adabelief: False                     ==
==            lr_dropout: n                         ==
==           random_warp: False                     ==
==      random_hsv_power: 0.0                       ==
==       true_face_power: 0.0                       ==
==      face_style_power: 0.0                       ==
==        bg_style_power: 0.0                       ==
==               ct_mode: rct                       ==
==              clipgrad: False                     ==
==              pretrain: True                      ==
==       autobackup_hour: 0                         ==
== write_preview_history: True                      ==
==           target_iter: 10001                     ==
==       random_src_flip: False                     ==
==       random_dst_flip: True                      ==
==            batch_size: 4                         ==
==             gan_power: 0.0                       ==
==        gan_patch_size: 16                        ==
==              gan_dims: 16                        ==
==                                                  ==
==------------------- Running On -------------------==
==                                                  ==
==          Device index: 0                         ==
==                  Name: NVIDIA GeForce GTX 750 Ti ==
==                  VRAM: 1.45GB                    ==
==                                                  ==
======================================================
Starting. Target iteration: 10001. Press "Enter" to stop training and save model.
[
[11:54:23][#003094][1766ms][2.5891][1.9999]

Pretraining

```


======



"G:\SAEHDBW\liae-udt-192-96-32-32-16-SAEHDBW_SAEHDBW_decoder.npy"
"G:\SAEHDBW\liae-udt-192-96-32-32-16-SAEHDBW_SAEHDBW_inter_B.npy"
"G:\SAEHDBW\liae-udt-192-96-32-32-16-SAEHDBW_SAEHDBW_encoder.npy"
"G:\SAEHDBW\liae-udt-192-96-32-32-16-SAEHDBW_SAEHDBW_inter_AB.npy"
"G:\SAEHDBW\liae-udt-192-96-32-32-16-SAEHDBW_SAEHDBW_data.dat"
"G:\SAEHDBW\liae-udt-192-96-32-32-16-SAEHDBW_SAEHDBW_src_dst_opt.npy"
234 МБ

"G:\SAEHDBW\liae-ud-192-128-32-32-16_SAEHDBW_src_dst_opt.npy"
"G:\SAEHDBW\liae-ud-192-128-32-32-16_SAEHDBW_encoder.npy"
"G:\SAEHDBW\liae-ud-192-128-32-32-16_SAEHDBW_data.dat"
"G:\SAEHDBW\liae-ud-192-128-32-32-16_SAEHDBW_decoder.npy"
"G:\SAEHDBW\liae-ud-192-128-32-32-16_SAEHDBW_inter_AB.npy"
"G:\SAEHDBW\liae-ud-192-128-32-32-16_SAEHDBW_inter_B.npy"
273 МБ



*** DF-UD ***

dfud-r96-32-32-12-bw_SAEHDBW_summary.txt -- 104 M
DF-UDT-256-96-32-32-16_SAEHDBW -- 281 M B: 4, 5 (OOM in minutes sometimes)
DF-UDT-R96-64-24-24-16-SAEHDBW_SAEHDBW -- 50 MB , train @mf 

"G:\SAEHDBW\df-udt-192-96-32-32-16_SAEHDBW_inter.npy"
"G:\SAEHDBW\df-udt-192-96-32-32-16_SAEHDBW_encoder.npy"
"G:\SAEHDBW\df-udt-192-96-32-32-16_SAEHDBW_data.dat"
"G:\SAEHDBW\df-udt-192-96-32-32-16_SAEHDBW_src_dst_opt.npy"
"G:\SAEHDBW\df-udt-192-96-32-32-16_SAEHDBW_decoder_dst.npy"
"G:\SAEHDBW\df-udt-192-96-32-32-16_SAEHDBW_decoder_src.npy"
285 МБ

"G:\SAEHDBW\ df-udt-192-128-48-32-16_SAEHDBW_decoder_src.npy"
"G:\SAEHDBW\ df-udt-192-128-48-32-16_SAEHDBW_encoder.npy"
"G:\SAEHDBW\ df-udt-192-128-48-32-16_SAEHDBW_inter.npy"
"G:\SAEHDBW\ df-udt-192-128-48-32-16_SAEHDBW_summary.txt"
"G:\SAEHDBW\ df-udt-192-128-48-32-16_SAEHDBW_data.dat"
"G:\SAEHDBW\ df-udt-192-128-48-32-16_SAEHDBW_src_dst_opt.npy"
"G:\SAEHDBW\ df-udt-192-128-48-32-16_SAEHDBW_decoder_dst.npy"
345 МБ

df-ud-192-128-48-48-16_SAEHDBW_summary
345 МБ
