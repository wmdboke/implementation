#### 1、templete理解

1. 模板本质是对类型参数话的一种工具

2. templete<class t> 跟 templete<typename T> 没有本质的区别

#### 2、定义

1. 函数模板
   
   template< class 形参名，class 形参名，......> 返回类型 函数名(参数列表)   { 函数体 }
   
   例：
   
   ```cpp
   templete<class T> 
   inline T func(T &a，T &b)
   ```

2. 类模板
   
   template< class 形参名，class 形参名，......> class 类名 {...};
   
   例：
   
   ```cpp
   template<class T> 
   class A{     
       public:         
           T g(T a,T b);         
           A(); 
   };
   template<class T> A<T>::A(){}
   
   template<class T> T A<T>::g(T a,T b){
        return a+b;
   }
   
   void main(){
        A<int> a;
        cout<<a.g(2,3.2)<<endl;
   }
   ```

3. 模板专门化
   
   有些特殊类型需要特殊处理，需要重载定义一次模板

        例如：

        template<class V> void swap(std::vector<V>& t1, std::vector<V>& t2) {

                t1.swap(t2);
       }.

    

   
