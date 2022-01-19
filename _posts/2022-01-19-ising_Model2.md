----
layout: single 

title: "Ising_Model-2"
----

###### Ising_Model 전체 코드

저번에 이어 이징모델 클래스를 업데이트 하였다.

```python
import numpy
from matplotlib import pyplot as plt
import matplotlib.cm as cm
import math

#격자크기를 받아 초기 이징모델 생성 클래스
class Ising2D:

    #멤버변수
    lattice_2D = [] 
    L=0
    T=5.0
    pos_x = 0 #lap_type의 x,y값을 멤버변수로 저장
    pos_y = 0
    transient = 1000

    def __init__(self,L): #생성자  , 인스턴스 변수
        Ising2D.L = L
        Ising2D.lattice_2D = Ising2D.gen_spinArray(Ising2D.lattice_2D,L)

    def gen_spinArray(array,L): #배열과 격자크기(L)를 입력받으면 L*L사이즈 랜덤스핀배열 리턴
        array = numpy.random.random((L,L))
        array = numpy.where(array < 0.5,-1,array)
        array = numpy.where(array > 0.5,1,array)
        return array

    def choose_lattice_position(void):
        Ising2D.pos_x = numpy.random.randint(0,Ising2D.L-1)
        Ising2D.pos_y = numpy.random.randint(0,Ising2D.L-1)

    #이미지 그래프 생성 함수
    def show_lattice(void):    #입력받은 스핀배열 그래프 출력
        plt.imshow(Ising2D.lattice_2D)
        plt.imshow(Ising2D.lattice_2D, cmap = cm.gray)
        plt.title('-1 : Black\n  1 : White')
        ax = plt.gca()
        ax.axes.xaxis.set_visible(False)
        ax.axes.yaxis.set_visible(False)
        plt.show()
        
    def energy_calc_single(x,y): # x,y=전자의 위치, 에너지 계산 함수 ()
        if y==(Ising2D.L-1): up=0
        else: up = y+1

        if y==0: down = (Ising2D.L-1)
        else: down = y-1

        if x==(Ising2D.L-1): right=0
        else: right = x+1

        if x==0: left = (Ising2D.L-1)
        else: left = x-1

        e = (-1) * Ising2D.lattice_2D[x][y] * \
                (Ising2D.lattice_2D[left][y] + \
                    Ising2D.lattice_2D[right][y] +\
                      Ising2D.lattice_2D[x][up] +\
                           Ising2D.lattice_2D[x][down])
        return e


    def energy_calc(x,y): #효율적인 위치계산
        e = Ising2D.lattice_2D[x][y]*(Ising2D.lattice_2D[x-1][y]+Ising2D.lattice_2D[x][y-1])
        return e


    def total_lattice_energy(void): #전체에너지 계산
        sum_energy = 0
        for i in range(Ising2D.L):
            for j in range(Ising2D.L):
                sum_energy += Ising2D.energy_calc(i,j)
        return sum_energy


    def test_flip(x,y,de): #뒤집기 테스트
        de = (-2)*Ising2D.energy_calc_single(x,y)
        if(de<0):return True
        elif (numpy.random.rand()<math.exp(-de/Ising2D.T)) : return True
        else: return False


    def flip(x,y):Ising2D.lattice_2D[x][y] = -Ising2D.lattice_2D[x][y]


    #랜덤 1000번 뒤집기
    def transient_results(void):
        for _ in range(Ising2D.transient):
            for _ in range((Ising2D.L)**2):
                Ising2D.choose_lattice_position(Ising2D.pos_x,Ising2D.pos_y)
                if Ising2D.test_flip(Ising2D.pos_x,Ising2D.pos_y): Ising2D.flip(Ising2D.pos_x,Ising2D.pos_y)
```

아직 최적화는 체크해보지 못했다.

우선 새로운 멤버변수를 추가하였다.

하나씩 체크해보며 복습해봐야겠다.







###### 수정된 멤버변수 

```python
 #멤버변수
    lattice_2D = [] 
    L=0
    T=5.0
    pos_x = 0 #lap_type의 x,y값을 멤버변수로 저장
    pos_y = 0
    transient = 1000
```

이전에 비해 멤버변수가 늘었다.

이징 모델에서 열수조 온도 설정을 위한 온도 상수 T

이전까지는 필요한 좌표값을 랜덤으로 받고 각 함수마다 좌표값을 대입할 생각을 했다. 근데 가만 생각해보니 리스트의 값만 불러올 수 있지 리스트의 x,y값은 내가 못꺼내온다... 따라서 새로운 멤버변수 pos_x,pos_y를 추가했고 이후에 이 두 값을 랜덤 설정해줄 함수를 만들었다.

transient은 아직 정확한 의미보다는 돌리는 규모를 나타낸다 정도로만 이해하고 넘어가보겠다.

###### 추가된 랜덤격자 선택함수

```python
def choose_lattice_position(void):
        Ising2D.pos_x = numpy.random.randint(0,Ising2D.L-1)
        Ising2D.pos_y = numpy.random.randint(0,Ising2D.L-1)
```

새롭게 추가한 함수다.

이 함수에서 랜덤하게 pos_x,pos_y 값을 설정한다. 격자 사이즈가 L이므로 그에 맞춰 범위안을 맞춰주었다.

*numpy.random.randint(a,b) : a~(b-1) 까지의 정수중 랜덤추출*







###### 최적화된 전체에너지 계산 함수

```python
def energy_calc(x,y): #효율적인 위치계산
        e = Ising2D.lattice_2D[x][y]*(Ising2D.lattice_2D[x-1][y]+Ising2D.lattice_2D[x][y-1])
        return e

def total_lattice_energy(void): #전체에너지 계산
	sum_energy = 0
    for i in range(Ising2D.L):
        for j in range(Ising2D.L):
            sum_energy += Ising2D.energy_calc(i,j)
    return sum_energy
```

전체에너지 계산만을 위한 두 함수다.

전체 헤밀토니안의 합을 이전의 energy_calc_single 함수로 계산하기엔 비효율적인 면이 있기 때문이다.

헤밀토니안을 계산할 때 이웃한 두 쌍의 전자스핀의 곱의 합을 구해야한다. 이 때 energy_calc_single 함수를 사용하면 순열합으로 계산되지만 실제 값은 Combination 합으로 가야한다. 따라서 다시 2로 나누어야 실제 헤밀토니안 합이 되는데 이전에 쓸모없는 계산을 2배나 한 셈이다.

따라서 새로운 계산함수 energy_calc은 한 위치에 대해 위,왼쪽과의 합만 계산한다. 

게다가 파이썬 특성상 [-1]위치는 맨 끝으로 이동하기 때문에 energy_calc_single 처럼 경계조건을 여러번 내세우지 않아도 자동으로 토러스 구조에 맞는 경계조건처럼 계산할 수 있다.

따라서 전체 헤밀토니안을 계산할 때는 energy_calc_single를 이용해 total_lattice_energy  함수로 전체에너지를 구한다.





생각해내는데는 몇일간 짬내가면서 떠올리고 코드를 짜는데 쓰고보면 왜이렇게 없어보일까 ㅠ
