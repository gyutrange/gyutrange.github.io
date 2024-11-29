---
layout: post
title: "2024 Autohack CTF Review"
date: 2024-11-29
author: GiuNash
tags: [CTF]
comments: true
toc: true
pinned: true
---
<style>
.caption {
    text-align: center;
    font-style: italic;
    color: gray;
}
</style>

### Autohack2024 개최

이번에 국민대학교 미래자동차 컨소시움에서 주관한 **Autohack 대회**에 다녀왔다.

#### 장소

장소는 대구 EXCO였는데, 왕복 9시간 걸렸다..

![coshow](https://github.com/user-attachments/assets/2538e137-78f9-480e-9ca2-d8223a9af55b)

늘 보던 CTF가 아니라, 실제 자동차를 대상으로 문제 풀이를 진행하는게 신기했다.

![autohack](https://github.com/user-attachments/assets/ec8cca71-0bb5-432d-806e-484903892a29)

여기서 문제를 풀거나, 자동차에 들어가는 CAN 데이터 등을 분석했고

자동차가 있는 대회장에 가서 직접 Injection을 진행하고, 로그를 수집했다.

#### 실제 자동차 기반의 대회

자동차는 Audi A5 모델과 Genesis g80으로 진행했다.

![realCar](https://github.com/user-attachments/assets/fe936b30-2a90-4ab8-97d1-f976f79a9bf9)
<div class="caption">뒤에 환기 시스템도 있어서 건물에 연기가 안 퍼진다.</div>

장비는 Kvaser, TOSUN을 사용했고, 차에 직접 연결해서 로그를 수집할 수 있었다.

![Audi](https://github.com/user-attachments/assets/8e2fc544-a037-4c82-8144-df47ac4e2d3e)
<div class="caption">~~~타고 도망가고 싶었다.~~~</div>

#### 후기

자동차 해킹은 이번이 처음이라 많이 당황했다.

CAN 패킷 분석, OBD-II, ECU의 이해 등등..

실제 자동차는 2초만 로깅을 돌려도 패킷이 2천개가 나온다.

그 중에 어떤 CAN ID의 어떤 byte가 자동차의 창문을 내리는 건지 등을 파악해야 했다.

대부분의 문제가 그렇게 공격 패킷을 차량에 보내서 창문 내리기, 차 잠그고 열기 등을 수행하는 것이었다.

결국 시동은 아무도 못 걸고 끝났지만..

![can](https://github.com/user-attachments/assets/5a1386d6-bb80-42d6-8605-8ef27f98996d)

힘들었지만 너무 재밌었다.

실제로 HackRF라는 장비를 사용해서 스마트 키의 주파수를 hijacking 하고 replay attack도 진행했고

스마트 키가 작동하지 않게끔 Jamming을 하는 문제도 있었다.

라즈베리파이를 이용해 차량에 연결된 스마트폰의 bluetooth 연결을 끊는 문제도 있었다.

마지막 날에는 IDS를 우회해서 공격하는 문제 세션도 있었다.

공격 바이트 찾으려고 하루 밤을 통으로 새웠다.

#### 수상

그래도 열심히 했더니 어찌어찌 수상은 했다!

잘하는 사람도 너무 많았고, 좋은 인연도 만들어서 좋았다.

시즌2 개최되면 한 번 더 나가야겠다.

![award](https://github.com/user-attachments/assets/5f1cd526-58aa-47b3-b025-7c565b0f415d)






