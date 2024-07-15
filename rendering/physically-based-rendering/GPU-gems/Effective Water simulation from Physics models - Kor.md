
이 노트에서 사용할 Wave models : 
1. vertex shader에서 표면의 높이와 방향을 계산하기 위해, 4개의 sin파를 합치는 모델
2. 같은 효과를 만들기 위해, 3개의 Gestner waves를 합치는 모델
3. (확장) 주기적 파동 함수의 합을 사용하여 , pixel shader에서 동적인 tiling bump map을 생성. (수면의 디테일을 개선)
---

## 1. Sum of sines approximation
water simulation을 위해서는 두 개의 surface 시뮬레이션이 필요합니다.
1. surface mesh의 기하적인 파동 시뮬레이션
2. surface mesh 상의 normal map 안의 잔물결(ripple)을 표현하는 시뮬레이션 

sine 파들의 합은 연속적이기 때문에, 모든 점에서의 물의 표면 방향과 높이를 묘사할 수 있습니다.
각 정점에 대한 수평적인 위치에 근거해서, sine 파의 합으로 표현된 함수로부터 샘플링을 진행하고, 연속적인 water surface 표현에 가까워지도록 tessellation을 최대한 진행해줍니다.

그 다음에는 texture 공간에서 추가적인 작업이 필요합니다. sine파들의 합으로 표현한 근사 함수로부터, 그 함수의 normal 벡터들을 샘플링하여 normal map을 생성할 수 있습니다.

각 프레임을 normal map으로 렌더링하는 것의 이점은 한정된 sine파들이 독립적으로 움직일 수 있도록 해준다는 것입니다.

## 1.1 wave parameters and model
![](../../../images/Pasted%20image%2020240702165314.png)

i번째 sine파는 다음과 같이 표현할 수 있습니다.
$$W_i(x,y,t)=A_i\times sin(D_i\cdot(x,y)\times w_i + t \times \varphi_i)$$
- $L$ :  특정 wave의 파장. 주파수 $w_i$ 와 $w_i = \frac{2}{L}$의 관계를 가집니다. (i번째 마루(crest) ~ i+1번째 마루 사이의 거리)
- $A$ : wave의 진폭. water plane으로부터 파의 마루까지의 높이를 나타냅니다.
- $S$ : wave의 속도. 초당 진행 방향으로 마루가 전진하는 거리로 나타냅니다. 구현 관점에서는 초기 위상인 $\varphi$로 표현하는 것이 간편합니다. ($\varphi = S \times 2/L$)
- $D$ : 파면(wavefront)과 직교하는 방향의 수평 벡터.

위의 단일 wave model 을 통해서, 전체 surface를 모델링 할 수 있습니다.
$$H(x,y,t)=\sum_i(A_i\times sin(D_i\cdot(x,y)\times w_i + t \times \varphi_i))$$

Scene을 동적으로 렌더링 하고 싶다면, 개별 sine wave의 파라미터들의 초기 값을 제약 조건 하에 랜덤으로 생성하면 됩니다.
또한, 지속적으로 하나의 wave의 효과를 서서히 사라지게 하고, 다른 파라미터 집합으로 다시 나타나게 함으로써 동적임을 더 강조합니다.

### 1.2 Normal and Tangents
 전체 surface에 대한 함수 $H(x,y,t)$가 명시적으로 표현되기 때문에, 임의의 점에 대해서 surface orientation을 직접적으로 계산할 수 있습니다.

임의의 시점 t에, total surface 상의 x,y 평면 좌표에 해당하는 3D 좌표는 다음과 같이 표현할 수 있습니다.
$$P(x,y,t) = (x,y,H(x,y,t))$$
이 점을 각 축에 대해서 편미분하면, binormal vector B와 tangent vector T에 해당하는 접선벡터를 구할 수 있습니다.
x와 y축은 서로 직교하기 때문에, 서로 다른 변수에 대한 편미분 값은 0이 됩니다.
$$B(x,y)=(\frac{\partial x}{\partial x},\frac{\partial y}{\partial x},\frac{\partial H(x,y,t)}{\partial x})=(1, 0, \frac{\partial H(x,y,t)}{\partial x})$$
$$T(x,y)=(\frac{\partial x}{\partial y},\frac{\partial y}{\partial y},\frac{\partial H(x,y,t)}{\partial y})=(0, 1, \frac{\partial H(x,y,t)}{\partial y})$$
벡터 B와 T의 관계는 아래의 그림처럼 나타납니다. (아래의 그림에서는 각각$\hat{b},\hat{t}$로 표현)
![](../../../images/Pasted%20image%2020240702172330.png)
선형대수의 외적 연산을 이용하면 두 벡터와 직교하는 제 3의 벡터를 얻을 수 있습니다.
그리고 그 벡터가 바로 구하고자 했던 surface normal(orientation)이 됩니다.
$$N(x,y) = B(x,y) \times T(x,y)$$
외적 연산 과정은 생략하고, normal 벡터는 다음과 같이 정리할 수 있습니다.
$$N(x,y)=(-\frac{\partial H(x,y,t)}{\partial x}, -\frac{\partial H(x,y,t)}{\partial y}, 1)$$

자 이제, 1.1에서 모델링 했던 $H(x,y,t)$를 편미분한 결과를 유도하기만 하면 됩니다!
일단 합의 미분은 미분의 합과 동일하므로 $H$ 편미분에 대한 식을 아래와 같이 전개합니다.

$$
\begin{eqnarray}
\frac{\partial H}{\partial x} = \sum\frac{\partial W}{\partial x}=\sum\frac{\partial (A_i\times sin(D_i\cdot(x,y)\times w_i + t \times \varphi_i))}{\partial x} \\
=\sum(w_i\times D_i.x\times A_i\times cos(D_i(x,y)\times w_i + t\times\varphi_i))
\end{eqnarray}
$$
미분 과정은 합성 함수 미분과 삼각 함수의 미분을 통해 해결했습니다.
또한 sin함수가 품고 있는 $D_i$에 대한 편미분은 미분 변수와 동일한 성분만 추출한다는 것을 확인해주세요. ($D_i.x$는 $D_i$의 $x$ 성분)

아직 위의 sine 파의 합 근사 모델링에는 약간의 결점이 있습니다.
실제 파도는 여러 개의 뾰족한 peak들을 가지고 더 넓은 바닥을 가집니다. 하지만 현재 우리의 수학적인 파도 모델은 부드럽고 완만한 경향이 있습니다.

이를 해결하기 위해, sine함수에 1을 더하여 최소값이 0이 되도록 offset을 변경하고, 지수 파라미터를 추가하여 뾰족함을 표현할 수 있도록 만들어 줍시다.

새로운 단일 wave는 다음의 등식을 만족합니다.
$$W_i(x,y,t)=2A_i\times \left(\frac{sin(D_i\cdot(x,y)\times w_i + t\times \varphi_i)}{2}\right)^k$$
이 새로운 단일 wave model을 다시 x,y로 편미분을 합니다. 결과식은 다소 복잡해보이지만, 미분 과정에서 사용되는 수학적 개념은 이전과 동일합니다.
$$
\begin{align}
\frac{\partial{W_i(x,y,t)}}{\partial{x}} &= kD_i.x\times w_i \times A_i \times \left(\frac{sin(D_i\cdot(x,y)\times w_i + t\times\varphi_i)+1}{2}\right)^{k-1}\times cos(D_i\cdot(x,y)\times w_i + t\times\varphi_i)
\end{align}
$$

- 지수 k 변화에 따른, wave 모양의 변화
![](../../../images/Pasted%20image%2020240702175201.png)

### 1.3 기하적인 waves
기하적인 waves를 위해서는 단순한 4가지 모델로 제한시킵니다.

#### a. Directional or Circular waves
- directional wave와 circular wave의 예시
![](../../../images/Pasted%20image%2020240703161540.png)


모델 중 하나인, directional wave는 다른 모델보다 shader instructions을 덜 요구합니다.
Directional wave에 대해서는 단일 wave 식에 포함되는 $D_i$ 방향 벡터가 wave의 생애주기 동안 고정적입니다. 반면, Circular wave는 각 정점에 대해서 방향을 계산해줘야합니다.  
 Circular wave $W_i$의 중심이 $C_i$라고 할 때,  $D_i$는 다음과 같이 쓸 수 있습니다.
 방향 벡터의 결과는 중심 벡터와 정점 벡터의 차이를 두 벡터 사이의 거리로 나눠주어 정규화한 결과와 같습니다.
 $$D_i(x,y) = \left( \frac{(x,y) - C_i}{|(x,y)-C_i|} \right)$$

#### b. Gerstner waves
더 효과적인 시뮬레이션을 하기 위해서는 wave들의 가파름을 제어할 수 있어야 합니다.
앞서 말했듯이, 기본 sine wave는 모든 구간에서 부드럽고 일정한 패턴을 보여줍니다. 하지만, 실제 거친 바다를 시뮬레이션 하고자 할 때는 파도가 튀어오르는 뾰족한 peak들을 표현할 수 있어야 합니다.

 이 효과를 나타내기 위해서 이전 섹션에서 새롭게 모델링한 지수적인 sine wave를 사용할 수도 있겠지만, 여기에서는 Gerstner wave를 사용하도록 하겠습니다.

Gerstner wave를 사용하는 이유는 개별 마루로 정점을 움직임으로써 더 날카로운 마루를 형성할 수 있기 때문입니다.
이는 surface에서 가장 날카로운 부분인 마루에 정점이 집중되도록 하고 싶을 때 딱 들어맞습니다.

- Gerstner wave 함수의 예시
![](../../../images/Pasted%20image%2020240703162849.png)

**The Gerstner wave function :**

$$
P(x,y,t) =
\begin{bmatrix}
x + \sum(Q_iA_i \times D_i.x \times cos(w_iD_i\cdot(x,y)+t\times\varphi_i)) \\
y + \sum(Q_iA_i \times D_i.y \times cos(w_iD_i\cdot(x,y)+t\times\varphi_i)) \\
\sum(A_i\sin(w_iD_i\cdot(x,y)+t\times\varphi_i))
\end{bmatrix}
$$

위 식에서 $Q_i$는 wave의 가파름을 조절하는 역할을 하는 파라미터입니다. $Q_i$가 0일 때는 정확히 기본 sine wave로만 구성된 모델 상 한 지점인 $(x,y,H(x,y,t))$와 동일합니다.
Gestner Wave를 사용하면 각 정점은 x,y 좌표에 더해진 cosine term에 따라 일정 범위를 수평 방향의 앞뒤로 진동하는 형태가 되고, 높이 성분의 sine에 따라 수직 방향으로 위 아래를 진동하는 형태가 됩니다. 즉 정점이 회전하는 형태를 표현함으로써 wave의 날카로움을 표현할 수 있게 되는 것입니다.
이때  $Q_i$를 너무 높게 잡는다면, 가장 날카로운 지점에서 롤로코스터와 같은 loop형태가 보일 수 있으므로 파라미터의 값을 적절하게 조절할 필요가 있습니다.

이전 방식과 동일하게 수평방향의 각 성분으로 편미분을 진행하여 binormal과 tangent 벡터를 구해내고, 두 벡터를 외적함으로써 surface orientation을 구할 수 있습니다. 
미분 결과는 살짝 지저분하지만, 과정 자체는 유사하므로 여기서는 생략하도록 하겠습니다.

- binormal vector B
  ![](../../../images/Pasted%20image%2020240703164946.png)
- tangent vector T
  ![](../../../images/Pasted%20image%2020240703164950.png)
- surface orientation N (normal vector)
  ![](../../../images/Pasted%20image%2020240703164954.png)
	여기서, $WA, S(), C()$는
	![](../../../images/Pasted%20image%2020240703165008.png)
	로 치환.

여기서 $Q_i$값에 따라 peak지점에서 loop가 왜 생기는지에 대해 엿볼 수 있습니다.
summation term이 1보다 크다면 수직 방향에 대한 normal성분이 아래를 향하는 경우가 생기게 됩니다. 이는 peak에서 surface의 orientation에 위를 향하도록 하고 싶은 우리의 기대에 반하는 경우이죠.
따라서 항상 summation term을 1보다 작게 유지함으로써 이 부정확한 현상을 방지할 수 있습니다.

### 파장과 속도

현실 세계에서의 분포를 따르지는 않고, 감당 가능할 정도의 wave들을 가지고 효과를 최대화하려고 합니다. 비슷한 길이의 wave들을 중첩시키는 것은 물 표면의 동적임을 강조하는데 도움이 됩니다.
따라서 파장의 중간 값을 선택하고, 그 중간 값의 절반과 두 배 사이에서 wave 길이의 랜덤 값을 추출할 것 입니다.
한 가지 기억해야할 것은 살아있는 wave의 파장을 바꿀 수는 없다는 것입니다. 만약 그렇게 할 수 있었더라면, 그 wave의 마루들은 origin으로부터 점점 멀어질 것이고 결과적으로 매우 부자연스러운 효과를 만들게 됩니다.
따라서 우리는 현재 파장의 평균값을 변화시킬 것이며, 기존의 어떤 wave가 사라짐에 따라 그 wave와 동일한 방향으로 바뀐 평균 파장값에 기반하여 새로운 wave를 생성할 것입니다.


파장이 주어졌을 때, 널리 알려진 물의 분산 관계(dispersion relation)에 따라 속도를 계산할 수 있습니다.
$$w = \sqrt{g\times \frac{2\pi}{L}}$$
, 여기서 $w$는 주파수 , g는 중력 그리고 L은 wave의 주기를 나타냅니다.

#### 진폭
진폭 계수를 어떻게 설정하느냐는 필요에 따라 달라질 수 있습니다. 
단순함을 위해서는 상수로 두면 됩니다.

#### 방향
wave가 움직이는 방향은 다른 파라미터들과 완전히 독립적입니다. 초기에는 사전에 설정된 바람 방향에 따라 고정시킬 수 있고, 점차 랜덤으로 방향을 바꿔나갈 수도 있을 것입니다.

### 1.4 wave 텍스쳐




---
## reference

- [GPU Gems 1 - Part 1 Natural effects, Chapter 1](https://developer.nvidia.com/gpugems/gpugems/part-i-natural-effects/chapter-1-effective-water-simulation-physical-models)
- images
	- https://en.m.wikipedia.org/wiki/File:Tangent_normal_binormal_unit_vectors.svg