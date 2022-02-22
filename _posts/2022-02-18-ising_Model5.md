---
layout: single 

title: "Ising_Model-3"
---



###### 전체코드

```python
from cmath import isnan
import numpy
from matplotlib import pyplot as plt
import matplotlib.cm as cm
import math
import matplotlib.animation as animation


#격자크기를 받아 초기 이징모델 생성 클래스
class Ising2D:


    #멤버변수
    lattice_2D = []
    T = 5.0
    pos_x = 0 #lap_type의 x,y값을 멤버변수로 저장
    pos_y = 0
    transient = 1000


    def __init__(self,L): #생성자  , 인스턴스 변수
        self.L = L
        if Ising2D.checkPositiveInteger(self.L):
            self.L = int(self.L)
            Ising2D.lattice_2D = Ising2D.gen_spinArray(Ising2D.lattice_2D,self)
        else:
            print("격자 사이즈가 양의 정수가 아닙니다. 클래스를 다시 생성하세요.")
            del(self)


    def __del__(self): #테스트 함수에서 여러 클래스를 생성하여 테스트 하기때문에 소멸자를 정의하기
        return


    def gen_spinArray(array,self): #배열과 격자크기(L)를 입력받으면 L*L사이즈 랜덤스핀배열 리턴
        array = numpy.random.random((self.L,self.L))
        array = numpy.where(array < 0.5,-1,array)
        array = numpy.where(array > 0.5,1,array)
        return array


    def checkPositiveInteger(num):
        num = str(num)
        if num.isdigit(): #숫자인지 판별
            if int(num) == float(num): #정수인지 판별
                if int(num) > 0 : #양수인지 판별
                    return True
        else: False


    def choose_lattice_position(self):
        Ising2D.pos_x = numpy.random.randint(0,self.L)
        Ising2D.pos_y = numpy.random.randint(0,self.L)


    #이징모델 클래스를 생성 후 그 클래스의 격자를 가지고 테스트할 함수
    #테스트 사항 : 균일한 확률로 랜덤하게 고르는지
    def Test_choose_lattice_position(self): 
        print("격자 위치선택 함수를 테스트할 횟수 : ", end = '')
        T = int(input())
        count_array = [[0]*self.L for _ in range(self.L)]
        for i in range(T):
            Ising2D.choose_lattice_position(self)
            count_array[Ising2D.pos_x][Ising2D.pos_y] += 1
        print(*count_array, sep = '\n')

        ave = int (T/self.L**2)
        for i in range(self.L):
            for j in range(self.L):
                count_array[i][j] -= ave
        print(*count_array, sep = '\n')
        

    #이미지 그래프 생성 함수
    #프레임당 mcs 횟수 입력받기 : mcs당 한번의 프레임을 하면 시각적인 변화가 너무 느림
    def show_lattice_Ani(self,mcs_per_frame):     #입력받은 스핀배열 그래프 출력
        fig, ax = plt.subplots()
        plt.title('-1 : Black\n  1 : White')
        ax = plt.gca()
        ax.axes.xaxis.set_visible(False)
        ax.axes.yaxis.set_visible(False)
        ims = []
        for i in range(Ising2D.transient):
            de=0
            for _ in range(mcs_per_frame):
                Ising2D.choose_lattice_position(self)
                if Ising2D.flip_test(Ising2D.pos_x,Ising2D.pos_y,de,self): Ising2D.flip(Ising2D.pos_x,Ising2D.pos_y)
            im = ax.imshow(Ising2D.lattice_2D,cmap = cm.gray ,animated=True)
            ims.append([im])
        ani = animation.ArtistAnimation(fig,ims,interval = 10)
        plt.show()
    

    def show_lattice(void):    #입력받은 스핀배열 그래프 출력
        fig = plt.figure()
        plt.imshow(Ising2D.lattice_2D)
        plt.imshow(Ising2D.lattice_2D, cmap = cm.gray)
        plt.title('-1 : Black\n  1 : White')
        ax = plt.gca()
        ax.axes.xaxis.set_visible(False)
        ax.axes.yaxis.set_visible(False)
        plt.show()


    def energy_calc_single(x,y,self): # x,y=전자의 위치, 에너지 계산 함수

        e = (-1) * Ising2D.lattice_2D[x][y] * \
                (Ising2D.lattice_2D[(x+self.L-1)%self.L][y] + \
                    Ising2D.lattice_2D[(x+self.L+1)%self.L][y] +\
                      Ising2D.lattice_2D[x][(y+self.L+1)%self.L] +\
                           Ising2D.lattice_2D[x][(y+self.L-1)%self.L])
        return e



    def total_lattice_energy(self): #전체에너지 계산 (파이썬 한정)
        sum_energy = 0
        for i in range(self.L):
            for j in range(self.L):
                sum_energy += (-1)*Ising2D.lattice_2D[i][j]*(Ising2D.lattice_2D[i-1][j]+Ising2D.lattice_2D[i][j-1])
        return sum_energy



    def flip_test(x,y,de,self): #뒤집기 테스트
        de = (-2)*Ising2D.energy_calc_single(x,y,self)
        if(de<0):return True
        elif (numpy.random.rand()<math.exp(-de/Ising2D.T)) : return True
        else: return False


    def flip(x,y):Ising2D.lattice_2D[x][y] = -Ising2D.lattice_2D[x][y]


    #랜덤 1000번 뒤집기
    def transient_results(self):
        de=0
        k=0  #테스트용 상수
        for _ in range(Ising2D.transient):
            for _ in range((self.L)**2):
                Ising2D.choose_lattice_position(self)
                if Ising2D.flip_test(Ising2D.pos_x,Ising2D.pos_y,de,self): Ising2D.flip(Ising2D.pos_x,Ising2D.pos_y);k+=1;print(k)
                # k+=1
                # print(k) 



    #전체에너지 계산 테스트
    def Test_single(self):
        for i in range(self.L):
            for j in range(self.L):
                print("({0},{1})에서 해밀토니안 에너지:".format(i+1,j+1),Ising2D.energy_calc_single(i,j,self))
        return



    #테스트 횟수를 입력받아 횟수만큼 랜덤배열이 출력되는지 테스트
    def Test_gen_spinArray(): 
        print("랜덤 스핀 배열을 테스트할 횟수 : " , end='')
        T = int(input())
        for _ in range(T):
            print("정사각형 격자 크기(L) 입력 : ", end='')
            size_L = input()
            testClass = Ising2D(size_L)
            if Ising2D.checkPositiveInteger(size_L) : print(testClass.lattice_2D, sep = '\n')



# # 전체에너지 계산 검산 테스트
# lat_test_calc_energy = Ising2D(2) 
# lat_test_calc_energy.Test_single() #1.싱글테스트 함수가 제대로 작동하는지 -> 실행시켜서 직접 계산이랑 맞는지 그자리에서 확인
# lat_test_calc_energy.show_lattice() #2.전부 계산해서 각각 맞게 나오는지 확인
# print(lat_test_calc_energy.total_lattice_energy()) #3. 전체 합이 전체에너지 계산함수 결과값과 같은지 확인


# # 애니메이션 실행
# lat_test_ani = Ising2D(30)
# for i in range(5):
#     lat_test_ani.T = 5-i
#     lat_test_ani.show_lattice_Ani(100)

# 스핀 배열 테스트
# Ising2D.Test_gen_spinArray()

lat = Ising2D(10)
lat.Test_choose_lattice_position()
```





###### 코드 리뷰

```python
 def show_lattice_Ani(self,mcs_per_frame):     #입력받은 스핀배열 그래프 출력
        fig, ax = plt.subplots()
        plt.title('-1 : Black\n  1 : White')
        ax = plt.gca()
        ax.axes.xaxis.set_visible(False)
        ax.axes.yaxis.set_visible(False)
        ims = []
        for i in range(Ising2D.transient):
            de=0
            for _ in range(mcs_per_frame):
                Ising2D.choose_lattice_position(self)
                if Ising2D.flip_test(Ising2D.pos_x,Ising2D.pos_y,de,self): Ising2D.flip(Ising2D.pos_x,Ising2D.pos_y)
            im = ax.imshow(Ising2D.lattice_2D,cmap = cm.gray ,animated=True)
            ims.append([im])
        ani = animation.ArtistAnimation(fig,ims,interval = 10)
        plt.show()
```

