----
layout: single 

title: "Ising_Model-3"
----

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



###### 플립테스트 함수, 플립 함수

```python
 def test_flip(x,y,de): #플립 테스트
        de = (-2)*Ising2D.energy_calc_single(x,y)
        if(de<0):return True
        elif (numpy.random.rand()<math.exp(-de/Ising2D.T)) : return True
        else: return False
 

 def flip(x,y):Ising2D.lattice_2D[x][y] = -Ising2D.lattice_2D[x][y]
```

이제 전체에너지가 아닌 몬테카를로 스텝을 위한 전자 하나의 플립 테스트가 필요하다. 이 과정은 에너지 최소화 원칙을 따른 것이다.

전자가 뒤집히기 위해선, 우선 에너지변화량이 음수이거나,

에너지 변화량이 양수라고 해도, 이 때 랜덤하게 생성된 난수가 볼츠만 분포에 의한 값보다 작으면 스핀이 뒤집힌다. 이는 엔트로피 최대화의 원칙에 따라 열수조에서 흡수된 에너지의 결과이다.

위 상황이 전부  적용되지 않을시에만 스핀은 뒤집히지 않는다.



플립 함수는 말 그대로 플립 동작만을 실행하는 함수이다.  스핀의 부호가 반대가 되므로 간단한 동작이다.







###### 뒤집기 함수

```python
#랜덤 1000번 플립(초기화)
    def transient_results(void):
        for _ in range(Ising2D.transient):
            for _ in range((Ising2D.L)**2):
                Ising2D.choose_lattice_position(Ising2D.pos_x,Ising2D.pos_y)
                if Ising2D.test_flip(Ising2D.pos_x,Ising2D.pos_y): Ising2D.flip(Ising2D.pos_x,Ising2D.pos_y)
```

랜덤하게 초기 스핀 난수를 생성했지만 순수한 랜덤이기에 자연 상태와는 거리가 있다.

따라서  transient_results 함수를 통해 격자 사이즈만큼 transient(=1000) 수를 플립테스트 후 플립한다.

좀 더 자연스러운 스핀 초기상태를 만들어서 시작하기 위함이다.