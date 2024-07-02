
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
---
## reference

- [GPU Gems 1 - Part 1 Natural effects, Chapter 1](https://developer.nvidia.com/gpugems/gpugems/part-i-natural-effects/chapter-1-effective-water-simulation-physical-models)
- images
	- https://en.m.wikipedia.org/wiki/File:Tangent_normal_binormal_unit_vectors.svg