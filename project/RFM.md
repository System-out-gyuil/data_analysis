# 🚩 Data analysis RFM project

## 미국 온라인 스토어 판매 데이터 분석

> https://www.kaggle.com/datasets/ytgangster/online-sales-in-usa

### 📌목차

- 1. features
- 2. 데이터 확인 및 전처리
- 3. RFM 스코어 구하기
- 4. RFM 스코어를 통해 고객 등급 구분
- 5. 판매 데이터 분석
- 6. 마케팅 전략 제안
- 7. 트러블 슈팅
- 8. 느낀점

### 1. features

order_date - 주문 날짜  
status - 배송상태  
item_id - 상품 ID  
qty_ordered - 주문 갯수  
price - 상품 가격  
category - 상품 카테고리  
payment_method - 결제 방식  
cust_id - 회원 ID  
Name Prefix - 호칭  
First Name - 이름  
Middle Initial - 중간 이름 이니셜  
Last Name - 성  
Gender - 성별  
age - 나이  
full_name - 이름  
Customer Since - 서비스 처음 이용 날짜, 가입날짜  
SSN - 미국식 주민등록 번호  
Phone No. - 폰번호  
Place Name - 지역  
County - 국가  
City - 도시  
State - 주  
Zip - 우편번호  
Region - 구역  
User Name - 닉네임

### 2. 데이터 확인 및 전처리

- 39개의 피쳐가 있습니다.
- 286392개의 행으로 이루어진 데이터입니다.

- 결측치는 없었습니다.
- 사용하지 않을 피쳐 7개를 제거하여 32개의 피쳐로 진행하였습니다.
  <details>
      <summary>피쳐 제거 코드보기</summary>
      
        ss_df = ss_df.drop(['sku', 'value', 'total', 'discount_amount', 'Discount_Percent', 'bi_st', 'ref_num'], axis=1)
  </details>

<br/>

- 복잡한 피쳐명을 변경하였습니다.

  <details>
      <summary>피쳐명 변경 코드보기</summary>

        ss_df.rename(columns={\
        'qty_ordered': 'quantity', 'payment_method': 'payment', 'Name Prefix':'name_prefix', 'First Name':'first_name'\
        , 'Middle Initial':'middle_initial', 'Last Name':'last_name', 'E Mail':'e_mail', 'Customer Since':'cust_since'\
        , 'Phone No.':'phone_num', 'Place Name':'place_name', 'User Name':'user_name'}, inplace=True)

  </details>

<br/>

### 3. RFM 스코어 구하기

#### Ⅰ. Recency구하기

- 데이터의 마지막 구매 날짜인 2021년 09월 30일을 기준으로 Recency를 계산하였습니다.

  <details>
    <summary>Recency 구하기 코드보기</summary>

      ss_df['date_point'] = pd.to_datetime(ss_df['order_date']).apply(lambda x: (pd.to_datetime('2021-09-30') - x).days)

  </details>

</br>

- 이 후 고객의 마지막 결제일을 통해 최근 구매날짜로 담아주었습니다.

  <details>
    <summary>Recency 구하기 코드보기</summary>

      ss_df.groupby('user_name')['date_point'].min().reset_index(name='Recency')

  </details>

</br>

#### Ⅱ. Frequency 구하기

- 각 고객별 구매 수량를 더하여 Frequenct를 구하였습니다.
  <details>
    <summary>Frequency 구하기 코드보기</summary>

      ss_df['user_name'].value_counts().reset_index(name='Frequency')

  </details>

</br>

#### Ⅲ. Monetory 구하기

- 각 고객별 구매 수량과 가격을 곱하여 Monetory를 구하였습니다.

  <details>
    <summary>Monetory 구하기 코드보기</summary>

      total_price = ss_df['quantity'] * ss_df['price']

      ss_df['total_price'] = total_price

      ss_df.groupby('user_name')['total_price'].sum().reset_index(name='Monetary')

  </details>

</br>

#### Ⅳ. RFM점수 스케일링

- 값의 비교를 위해 MinMaxScaler를 통해 0~1 사이의 값으로 스케일링해주었습니다.

  <details>
    <summary>스케일링 코드보기</summary>

      scaler = MinMaxScaler()

      ss_rfm[['Recency', 'Frequency', 'Monetary']] = scaler.fit_transform(ss_rfm[['Recency', 'Frequency', 'Monetary']])

  </details>

</br>

- 이 후 최근 구매 일 기준으로 가장 최근에 구매한 고객의 점수가 0으로 가장 낮게 되어있어서  
  1에 Recency점수를 빼주어서 가장 최근에 구매한 고객의 점수가 가장 높게 나타나게 하였습니다.

#### Ⅴ. 최종 RFM 점수 구하기

- Recency와 Frequency, Monetory를 더하여 RFM스코어를 만들어줍니다.

  <details>
      <summary>RFM합산 코드보기</summary>

      ss_rfm['TotalScore'] = ss_rfm[['Recency', 'Frequency', 'Monetary']].sum(axis=1)

      ss_rfm

    </details>

</br>

- 아래와같이 각 고객별 RFM 스코어가 나왔습니다.

</br>

  <table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>user_name</th>
      <th>Recency</th>
      <th>Frequency</th>
      <th>Monetary</th>
      <th>TotalScore</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>aaabell</td>
      <td>0.302198</td>
      <td>0.001982</td>
      <td>0.007803</td>
      <td>0.707587</td>
    </tr>
    <tr>
      <th>1</th>
      <td>aaacree</td>
      <td>0.552198</td>
      <td>0.000000</td>
      <td>0.000062</td>
      <td>0.447864</td>
    </tr>
    <tr>
      <th>2</th>
      <td>aaagin</td>
      <td>0.760989</td>
      <td>0.000396</td>
      <td>0.001256</td>
      <td>0.240663</td>
    </tr>
    <tr>
      <th>3</th>
      <td>aaallie</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000360</td>
      <td>1.000360</td>
    </tr>
    <tr>
      <th>4</th>
      <td>aaapperson</td>
      <td>0.755495</td>
      <td>0.001189</td>
      <td>0.009196</td>
      <td>0.254890</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>64001</th>
      <td>zzmiers</td>
      <td>0.824176</td>
      <td>0.000000</td>
      <td>0.000066</td>
      <td>0.175890</td>
    </tr>
    <tr>
      <th>64002</th>
      <td>zzrapoza</td>
      <td>0.695055</td>
      <td>0.000000</td>
      <td>0.000065</td>
      <td>0.305010</td>
    </tr>
    <tr>
      <th>64003</th>
      <td>zzrascoe</td>
      <td>0.513736</td>
      <td>0.000000</td>
      <td>0.000870</td>
      <td>0.487133</td>
    </tr>
    <tr>
      <th>64004</th>
      <td>zzsotelo</td>
      <td>0.763736</td>
      <td>0.000000</td>
      <td>0.000033</td>
      <td>0.236296</td>
    </tr>
    <tr>
      <th>64005</th>
      <td>zzstansfield</td>
      <td>0.376374</td>
      <td>0.000396</td>
      <td>0.000092</td>
      <td>0.624114</td>
    </tr>
  </tbody>
</table>

</br>

### 4. RFM스코어를 통해 고객 등급 구분

- 각 고객별 RFM 스코어를 일정 구간별로 나누어 다섯개의 등급으로 나누어 주었습니다.

  <details>
    <summary>고객 등급 나누기 코드보기</summary>

      l1, l2, l3, l4 = np.percentile(ss_rfm_df['TotalScore'], [20, 40, 70, 90])

      def get_level(x):
        if x <= l1:
            return 5
        if x <= l2:
            return 4
        if x <= l3:
            return 3
        if x <= l4:
            return 2
        return 1

      ss_rfm_df['Level'] = ss_rfm_df['TotalScore'].apply(get_level)

      ss_rfm_df['Level'] = ss_rfm_df['Level'].replace([5, 4, 3, 2, 1],
                                                ['Bronze', 'Silver', 'Gold', 'Diamond', 'VIP'])

  </details>

  </br>

- 아래 시각화와 같이 골드등급이 가장 많고 VIP등급이 가장 적은 분포로 등급을 매겨주었습니다.

  <img src="https://github.com/System-out-gyuil/System-out-gyuil/assets/120631088/3ca126a3-c170-46ff-ac7e-bc8ac6fe0697">

  </br>

### 5. 판매 데이터 분석

#### Ⅰ. 각 나이대별 등급 분포

   <img src="https://github.com/System-out-gyuil/System-out-gyuil/assets/120631088/0d26ea6f-7a00-43a5-a1d3-f3d96fb77c1d">

- 40대의 브론즈 등급이 상대적이게 많으며, 50대의 VIP회원이 가장 많은 모습입니다.

#### Ⅱ. 각 카테고리 별 등급 분포

   <img src="https://github.com/System-out-gyuil/System-out-gyuil/assets/120631088/75815887-9118-446b-af8f-708be4210c1e">

- 브론즈 - men's fashion, appliances, mobiles & tablets, entertainment 분야에서 비중이 높습니다.
- 골드 - 대부분의 카테고리에서 높은 수치를 보이며, 특히 mobiles & tablets, women's fashion, others에서 높은 수치를 보입니다.
- 다이아 - beauty & grooming와 superstore, women's fashion, school & education에서 상대적으로 높은 수치를 보입니다.
- VIP - health & sports에 굉장한 관심을 보이고있습니다.

#### Ⅲ. 각 나이대별 카테고리 분포

  <img src="https://github.com/System-out-gyuil/System-out-gyuil/assets/120631088/8412c3fb-2e9c-493e-bee0-b383ff49b3de">

- 10대 - health & sports 부분에 특히 관심이 없습니다. (건강에 관심 없을 나이)
- 20대 - entertainment 가 상대적으로 높은 수치를 보입니다. (유흥거리를 찾는 듯 함)
- 30대 - 20대와 달리 entertainment보다 superstore를 더 자주 이용하는 모습입니다. (더 실용적인걸 찾게되는듯 함)
- 40대 - 크게 특이점은 없지만 상대적이게 mobile & tablets수치가 낮은편입니다.
- 50대 - 전체 나이대중 가장 높은 수치를 나타내며, 갑자기 health & sports에 엄청난 관심을 보이며, soghaat(선물)의 수치가 상대적으로 높아진 모습입니다. (건강에 관심을 갖고 주변사람을 챙기는 듯 함)
- 60대 - 크게 특이점이 없지만 books의 수치가 조금이나마 높아진 모습입니다.
- 70대 - health & sports가 상대적이게 낮아졌으며, 특히 패션부분에 굉장히 관심이 없어진 모습입니다.

#### Ⅳ. 각 카테고리별 주문 취소 분포

  <img src="https://github.com/System-out-gyuil/System-out-gyuil/assets/120631088/318143be-24b5-4399-868c-4edf9ae57edf">

- mobiles & tablets의 주문 취소가 전체적이게 아주 많고
- gold등급의 others 주문 취소가 굉장히 높은 모습이다.

#### Ⅴ. 각 카테고리별 환불 분포

  <img src="https://github.com/System-out-gyuil/System-out-gyuil/assets/120631088/d1af03fe-bcc7-4387-818f-e8c144ece3ed">

- 환불또한 mobiles & tablets의 수치가 높으며
- men's fashion의 환불이 굉장히 많고, women's fashion또한 꽤 높은 수치를 보이고 있다.

#### Ⅵ. 가장 많은 구매를 한 상위 다섯개 지역

  <img src="https://github.com/System-out-gyuil/System-out-gyuil/assets/120631088/cce7e2fb-fa6f-4d7e-96e2-c64f13459100">

- Dekalb지역이 정말 특이하게 VIP만 굉장히 많다
- 반대로 Washington은 VIP가 없는 모습이다.

### 6. 마케팅 전략 제안

- 가장 인기가 많은 Mobiles & Tablets가 브론즈 등급과 골드 등급이 상대적이게 많으며  
  20대, 30대 고객과 50대 60대 고객이 가장 많기에 상대적 으로 가격에 예민한 20대와 30대를 겨냥하여 할인행사를 시행하고  
  상대적으로 나이대가 좀 있는 50대와 60대를 겨냥하여 더 접근성을 높이기 위하여 상품의 설명을 쉽게 도와주거나 홈쇼핑 등 다양한 방법을 고려해볼 필요가 있어보임.

- 50대의 Health & Sports 와 soghaat(선물)의 관심이 굉장히 많으며, 50대의 VIP가 가장 많고,  
  카테고리 중 Health & Sports의 VIP가 가장 많다는 점을 고려하여 Health & Sports의 제품을 선물해주면 할인을 해주는 등  
  Health & Sports와 soghaat를 엮어서 마케팅 하는 전략이 좋을 것 같다.

- dekalb 지역의 VIP만 굉장히 많기때문에 해당 지역에서의 신규 회원 유입을 위한 홍보를 하고, 기존 VIP 고객 유치를 위해 VIP 등급의 혜택을 주는 전략이 좋을 것 같습니다.

- 취소와 환불의 전체 수치가 굉장히 높은데 취소 및 환불의 사유등을 조사하여 취소와 환불을 하지 않도록 만드는 전략을 찾아 볼 필요가 있을 것 같다.

### 7. 트러블 슈팅

- RFM스코어에 가중치를 주지 않고 분석하는것을 배워서 기본 RFM스코어 그대로 사용하였던 문제가 있었다.

- 따라서 해당 데이터의 주제에 맞게 RFM에 가중치를 주고싶은 부분에 가중치를 주고 진행해보았습니다.

### 8. 느낀점

- 또한 고객들의 정보를 통해 어떤 계층(수입 등)인지 알 수 있다면 훨씬 디테일한 분석이 가능했을 것 같다.

- 도메인 지식의 부족으로 인하여 미국의 각 지역별 특징을 잘 몰라서 분석에 어려움이 있었다.
