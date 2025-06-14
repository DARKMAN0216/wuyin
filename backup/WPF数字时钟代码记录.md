
**ViewBox**
数字时钟逻辑类

    public class DigitalClockViewClock : BaseViewModel, IDisposable
    {
    private DispatcherTimer _timer;
    private string _currentDateTimeString;
    private string _currentDayOfWeekString;
    // 暴露给 View 绑定的属性
    public string CurrentDateTimeString
    {
        get => _currentDateTimeString;
        set => Set(ref _currentDateTimeString, value);
    }
    public string CurrentDayOfWeekString
    {
        get => _currentDayOfWeekString;
        set => Set(ref _currentDayOfWeekString, value);
    }
    public DigitalClockViewClock()
    {
        // 初始化定时器
        _timer = new DispatcherTimer();
        _timer.Interval = TimeSpan.FromSeconds(1); // 每秒更新一次
        _timer.Tick += Timer_Tick
        // 立即更新一次显示
        UpdateDateTime();
        // 启动定时器
        _timer.Start();
    }

    private void Timer_Tick(object sender, EventArgs e)
    {
        // 定时器触发时更新 ViewModel 的属性
        UpdateDateTime();
    }

    private void UpdateDateTime()
    {
        DateTime now = DateTime.Now;

        // 更新日期和时间 (格式: YYYY-MM-DD HH:mm)
        CurrentDateTimeString = now.ToString("yyyy-MM-dd HH:mm");

        // 更新星期几 (格式: 星期一, 星期二, ...)
        CurrentDayOfWeekString = now.ToString("dddd");
    }

    // 实现 IDisposable 接口，用于清理资源（停止定时器）
    public void Dispose()
    {
        Dispose(true);
        GC.SuppressFinalize(this);
    }

    protected virtual void Dispose(bool disposing)
    {
        if (disposing)
        {
            // 停止定时器并移除事件处理器
            if (_timer != null)
            {
                _timer.Stop();
                _timer.Tick -= Timer_Tick;
                _timer = null;
            }
        }
    }

    // Finalizer (如果 Dispose 没有被调用，垃圾回收器会尝试调用它)
    ~DigitalClockViewClock()
    {
        Dispose(false);
    }
    }

**MainWindowView.xaml.cs**
数字时钟展示及回收

    public partial class MainWindowView: Window
    {
    private DigitalClockViewClock _digitalClockViewClock;
    public MainWindowView()
    {
        InitializeComponent();
        _digitalClockViewClock = new DigitalClockViewClock();

        // 将 ViewModel 设置为 DataContext
        this.DataContext = _digitalClockViewClock;

        // 在窗口关闭时处理 ViewModel 的 Dispose
        this.Closed += MainWindowView_Closed;
    }

    private void MainWindowView_Closed(object sender, EventArgs e)
    {
        // 在窗口关闭时调用 ViewModel 的 Dispose 方法
        if (_digitalClockViewClock != null)
        {
            _digitalClockViewClock.Dispose();
            _digitalClockViewClock = null;
        }
    }
    }

**HGDigitalTwinWindow.xaml**
数字时钟xaml展示

    <StackPanel Grid.Column="2" Margin="0,0,10,0" Orientation="Horizontal">
    <TextBlock Text="{Binding CurrentDateTimeString}" Foreground="#D4F4F1" FontSize="25"/>
    <TextBlock Text="{Binding CurrentDayOfWeekString}" Foreground="#D4F4F1" HorizontalAlignment="Right" FontSize="25" Margin="10,0"/>
    </StackPanel>