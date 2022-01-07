### Raw Image

가공되지 않은 (전반적인 프로세싱 과정 중 아직 압축 등의 변환이 되지 않은) 순수한 데이터. Digital Signal Processing이 이루어지지 않은 상태로 저장된 원본 데이터로 pattern 변환이 이루어지지 않은 것.

<br/>

### Pixel

1200만 화소 카메라 = 1200만 개의 셀(이미지 센서의 구성 요소 단위)로 구성된 이미지 센서가 탑재된 카메라. 촬영한 피사체의 R, G, B 색을 감지할 수 있는 셀이 1200만 개.

<br/>

### Filter & Pattern

색상을 입히는 것. 이미지 센서에는 Color Filter Array가 존재하고, 어떤 CFA를 사용하느냐에 따라 생성되는 색 공간의 타입이 달라짐. Bayer, CYYM, RGBE 등이 많이 사용되는 필터. 

그 중 raw Bayer pattern 데이터를 그대로 출력하면 카메라의 데이터 전송량을 크게 줄일 수 있음. 카메라에서 촬영된 이미지는 케이블을 통해 외부로 전송됨. 이 때 케이블의 데이터 전송용량(bandwidth)에는 한계가 있기 때문에 고해상도로 이미지를 내보내려면 FPS가 떨어지는 단점이 있음. (USB 2.0은 초당 30MB/s가 한계)

카메라에서 1024x1024 해상도의 RGB 칼라 영상을 내보내는 경우 이미지 한 장당 3 MB의 데이터 전송이 필요하지만, raw Bayer pattern 데이터를 그대로 내보내면 색상에 대한 손실 없이 1 MB의 데이터만 전송하면 되기 때문에 FPS를 3배 정도 높일 수 있음.

Bayer filter를 적용하여 나온 결과 이미지를 Bayer pattern이라고 함. BGGR 순서로 배치된 2x2 배열이 반복적으로 구성됨. G를 2번 사용하는 이유는

1. 자연계에 존재하는 가장 많은 색상
2. G 값의 비중이 휘도(밝기)에 가장 큰 영향을 줌
3. 인간의 눈은 초록색에 민감

![image](https://user-images.githubusercontent.com/44194558/148499298-250313fb-7602-4a6f-a8f1-a24d13832f40.png)

다만, 하나의 셀은 RGB 필터 중 하나로만 결합되기 때문에 하나의 이미지 픽셀은 하나의 색만 감지한다. 따라서 각 픽셀에서 없는 색상은 주변의 픽셀을 이용하여 생성하게 된다. 실제 이미지 센서는 각 픽셀에서 RGB 중 어느 한 색만을 감지할 수 있지만, 우리가 보는 카메라 영상에서는 각 픽셀마다 RGB 전체 색상을 보여줌.

인간이 실제로 보는 이미지 영상은 모두 주변 픽셀을 통해 인위적으로 생성된 결과이며, 이러한 과정을 **Demosaicing**이라고 함. (주변의 픽셀 값을 이용해서 최대한 가깝게 복원)

![image](https://user-images.githubusercontent.com/44194558/148499931-9df394e6-1ca2-4f16-9230-417b6ac5f794.png)

 - 원본 이미지

<br/>


![image](https://user-images.githubusercontent.com/44194558/148499980-bb8ce941-792c-45db-8b23-b3d0a21a30d5.png)
 
 - Bayer filter 이미지

<br/>

![image](https://user-images.githubusercontent.com/44194558/148499835-73388c54-976a-4e64-b159-7ecbf52c1a04.png)

 - 전체 픽셀에 대해 bayer filter 이미지를 구한 후, 각 픽셀과 주변의 픽셀 값을 이용해서 원본 이미지를 복구

<br/>

![image](https://user-images.githubusercontent.com/44194558/148500141-8e256735-55a9-4eea-943e-17d31771b77a.png)

 - 원본 이미지처럼 색이 복원된 이미지

<br/>

### Demosaicing

![image](https://user-images.githubusercontent.com/44194558/148500588-a633473d-7fe6-4e98-9f53-a41a5d5be479.png)

오른쪽은 원본 이미지에 필터를 적용시킨 결과. G가 2번 샘플링되기 때문에 전체적으로 초록색에 가깝게 보임.

원본 이미지가 픽셀당 24bit(3 x 8bit)의 데이터를 가지는데 비해, 필터를 적용한 결과 이미지는 하나의 픽셀 당 RGB 중 1개 채널의 데이터(8bit)만을 가지고 나머지는 모두 0이 된다. 즉 필터를 적용하면 실질적인 데이터의 양이 1/3로 줄어들게 됨.

실제로 필터를 적용시킨 결과 이미지를 각 채널 별로 분리해서 시각화하면 픽셀 값이 비어있음을 확인할 수 있다.

![image](https://user-images.githubusercontent.com/44194558/148500932-494b39ec-b291-4af3-8cc8-00d67b026698.png)

하지만 줄어든 데이터만을 가지고 원래의 색상 이미지를 보기 위해서는 각 픽셀이 가진 데이터에 해당하는 RGB 채널이 어떤 것인지 알아야 하기 때문에 mosaicing에 사용된 RGB 패턴을 알아내야 한다.

다음은 주변의 데이터를 이용하여 평균을 구하는 방식으로 interpolation을 수행하여 demosaicing을 수행한 결과.


![image](https://user-images.githubusercontent.com/44194558/148501277-dcc57be6-5104-42c7-be10-53145a4f070e.png)

 - interpolation 수행 결과 mosaicing된 이미지 특유의 격자 무늬가 사라지고 원본에 비해 약간 흐려짐
 - 실제로 아래와 같이 각각의 채널에서 빈 공간이 모두 메워진 것을 확인할 수 있음

![image](https://user-images.githubusercontent.com/44194558/148502495-429ea381-3ff2-464a-8115-3ea4ebef0b4b.png)

위의 interpolation은 RGB 채널 간의 correlation을 전혀 고려하지 않은 연산 방식. RGB 간에는 중복되는 파장이 존재하기 때문에 중복되는 정보가 존재하게 됨. 채널 간 상관성을 이용하여 demosaicing된 RGB 이미지의 어떤 한 채널에서의 빈 정보를 다른 채널의 정보에서 어느 정도 가져올 수 있음.

![image](https://user-images.githubusercontent.com/44194558/148502770-42022dd3-2c4e-4920-aca9-2d49495ae843.png)

 - G을 먼저 interpolate한 다음, R/B와 중복되는 G의 정보를 각각의 채널에서 뺌
 - G의 정보를 뺀 결과를 interpolation한 다음, 뺄셈했던 G듸 정보를 더해줌
 - R의 빈 공간에서의 정보 중 G와 중복되는 정보는 원본과 동일하게 됨

중복되는 정보의 처리는 YUV 색 공간에서 시행됨. RGB와 YUV간의 변환에는 다음의 행렬이 사용됨.


![image](https://user-images.githubusercontent.com/44194558/148503309-d1e2d4ab-f3bb-45ed-b9db-2a5ea3aded7d.png)