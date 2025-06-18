**核心目的**
WPF中用户的界面元素（例如，按钮`button`，菜单项`MenuItem`等）通常需要触发ViewModel中某个逻辑或操作，WPF提供了一种标准的方式来实现这种交互，即通过绑定UI元素的`command`属性到ViewModel中实现`System.Windows.Input.ICommand`接口的属性。
`ICommand`提供了三个接口方法：
`void Execute(object parameter)`:当UI元素被触发时，会调用此方法执行命令逻辑`parameter`允许传递额外数据。
`bool CanExecute(object parameter)`: 这个方法用于判断命令当前是否可以执行。WPF UI 元素会调用此方法来决定是否启用或禁用自身（例如，如果 CanExecute 返回 false，按钮通常会被禁用），parameter 参数同样允许传递数据。
`event EventHandler CanExcuteChangerd`:当命令的可执行状态（即 CanExecute 方法的返回值）可能发生变化时，命令需要触发这个事件来通知所有订阅者（主要是绑定的 UI 元素），让它们重新查询 CanExecute 方法以更新其启用/禁用状态。

就像 `INotifyPropertyChanged` 一样，为 ViewModel 中的每一个命令都创建一个单独的类来实现 `ICommand` 接口是非常重复和低效的。你需要为每个命令类编写 Execute 和 CanExecute 的具体逻辑，并且更重要的是，你需要管理` CanExecuteChanged `事件的触发时机，这通常涉及到监听 ViewModel 中影响命令可执行状态的属性变化。

**RelayCommand 的作用**：
RelayCommand (或 DelegateCommand) 就是为了解决这个问题而设计的。它是一个通用的 ICommand 实现类，它不包含具体的业务逻辑，而是将 Execute 和 CanExecute 的调用转发 (Relay) 给在创建 RelayCommand 实例时传入的委托（通常是 Action 和 Func<bool>）。这样，ViewModel 只需要提供这些委托方法，而不需要为每个命令编写一个完整的 ICommand 类。

    /// ICommand 接口的通用实现，将执行和可执行逻辑委托给 Action 和 Func。
    public class RelayCommand : ICommand
    {
        // 存储执行命令的委托 (Action<object> 接受一个 object 参数)
        private readonly Action<object> _execute;

        // 存储判断是否可执行的委托 (Func<object, bool> 接受一个 object 参数并返回 bool)
        private readonly Func<object, bool> _canExecute;

        /// 当命令的可执行状态可能发生变化时触发。
        /// 通常关联到 CommandManager.RequerySuggested 事件，
        /// 这是 WPF 提供的一种机制，用于在某些系统事件发生时提示命令重新评估其状态。
        public event EventHandler CanExecuteChanged
        {
            add { CommandManager.RequerySuggested += value; }
            remove { CommandManager.RequerySuggested -= value; }
        }

        /// 创建一个新的 RelayCommand 实例。
        public RelayCommand(Action<object> execute, Func<object, bool> canExecute = null)
        {
            // 确保执行委托不为空
            _execute = execute ?? throw new ArgumentNullException(nameof(execute));
            _canExecute = canExecute;
        }

        /// 判断命令当前是否可以执行。
        public bool CanExecute(object parameter)
        {
            // 如果没有提供 _canExecute 委托，则命令总是可执行 (返回 true)。
            // 如果提供了 _canExecute 委托，则调用它并返回其结果。
            return _canExecute == null || _canExecute(parameter);
        }

        /// 执行命令。
        public void Execute(object parameter)
        {
            // 调用存储的执行委托。
            _execute(parameter);
        }

        // --- 可选的重载构造函数：处理不带参数的命令 ---
        /// 创建一个新的 RelayCommand 实例，用于不带参数的命令。
        public RelayCommand(Action execute, Func<bool> canExecute = null)
            // 调用主构造函数，将无参数委托包装成带 object 参数的形式
            : this(p => execute(), p => canExecute?.Invoke() ?? true)
        {
            if (execute == null) throw new ArgumentNullException(nameof(execute));
        }
}