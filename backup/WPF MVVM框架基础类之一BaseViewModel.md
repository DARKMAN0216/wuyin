**核心目的**
当ViewModel数值发生变化时，绑定对应属性的View也要发生变化，两者之间的通知机制是通过实 `System.ComponentModel.INotifyPropertyChanged`  接口来完成的。
`INotifyPropertyChanged`接口中包含一个唯一事件`public event PropertyChangedEventHandler PropertyChanged;`每当ViewModel 的某个属性值改变时，你需要触发这个 PropertyChanged 事件，并传递一个 PropertyChangedEventArgs 对象，其中包含发生变化的属性的名称（字符串），手动实现时需要将每个字段都进行检查，更新以及触发事件，编写繁琐重复且容易出错。
BaseViewModel的主要作用就是解决上述重复性问题而诞生的，通过提供通用方法简化属性的设置和通知。
**具体实现**

     public class BaseViewModel : INotifyPropertyChanged
     {

        public event PropertyChangedEventHandler PropertyChanged;

        protected void OnPropertyChanged([CallerMemberName] string propertyName = null)
        {
            PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
        }

        protected bool SetProperty<T>(ref T field, T value, [CallerMemberName] string propertyName = null)
        {
            if (EqualityComparer<T>.Default.Equals(field, value))
            {
                return false; 
            }
            field = value;
            OnPropertyChanged(propertyName);
            return true;
        }
    }
解析：依旧通过实现INotifyPropertyChanged的PropertyChangedEventHandler 来处理双向数据同步，增加两个方法，一个通过CallerMemberName（C#5.0引入的特性，当被调用的方法存在CallerMemberName标记时，编译器自动将调用方的成员名称作为参数值传递进入）消除手动输入属性名不准确的问题；另一个用于批量监听值传递，如果传入属性值发生改变则赋值，未改变则下一循环。