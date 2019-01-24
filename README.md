# Save-the-Humans (Desktop App)

XAML to C# - Desktop App game

/*C# Visual Studio 2017 on windows 10*/

    using System;
    using System.Collections.Generic;
    using System.Linq;
    using System.Text;
    using System.Threading.Tasks;
    using System.Windows;
    using System.Windows.Controls;
    using System.Windows.Data;
    using System.Windows.Documents;
    using System.Windows.Input;
    using System.Windows.Media;
    using System.Windows.Media.Imaging;
    using System.Windows.Navigation;
    using System.Windows.Shapes;

    using System.Windows.Media.Animation;
    using System.Windows.Threading;

    namespace Save_the_Humans
    {
        /// <summary>
        /// Interaction logic for MainWindow.xaml
        /// </summary>
        public partial class MainWindow : Window
        {
            Random random = new Random();
            DispatcherTimer enemyTimer = new DispatcherTimer();
            DispatcherTimer targetTimer = new DispatcherTimer();
            bool humanCaptured = false;

        public MainWindow()
        {
            InitializeComponent();

            enemyTimer.Tick += EnemyTimer_Tick;
            enemyTimer.Interval = TimeSpan.FromSeconds(2);

            targetTimer.Tick += TargetTimer_Tick;
            targetTimer.Interval = TimeSpan.FromSeconds(.1);
        }

        private void TargetTimer_Tick(object sender, EventArgs e)
        {
            progressBar.Value += 1;
            if (progressBar.Value >= progressBar.Maximum)
                EndTheGame();
        }

        private void EndTheGame()
        {
            if (!playArea.Children.Contains(gameOverText))
            {
                enemyTimer.Stop();
                targetTimer.Stop();
                humanCaptured = false;
                startButton.Visibility = Visibility.Visible;
                playArea.Children.Add(gameOverText);
                    
            }
        }

        private void EnemyTimer_Tick(object sender, EventArgs e)
        {
            AddEnemy();
        }

        private void StartButton_Click(object sender, RoutedEventArgs e)
        {
            StartGame();
        }

        private void StartGame()
        {
            human.IsHitTestVisible = true;
            humanCaptured = false;
            progressBar.Value = 0;
            startButton.Visibility = Visibility.Collapsed;
            playArea.Children.Clear();
            playArea.Children.Add(target);
            playArea.Children.Add(human);
            enemyTimer.Start();
            targetTimer.Start();
        }

        private void AddEnemy()
        {
            ContentControl enemy = new ContentControl();
            enemy.Template = Resources["EnemyTemplate"] as ControlTemplate;
            AnimateEnemy(enemy, 0, playArea.ActualWidth - 100, "(Canvas.Left)");
            AnimateEnemy(enemy, random.Next((int)playArea.ActualHeight - 100), 
                random.Next((int)playArea.ActualHeight - 100), "(Canvas.Top)");
            playArea.Children.Add(enemy);

            enemy.MouseEnter += Enemy_MouseEnter;
                }

        private void Enemy_MouseEnter(object sender, MouseEventArgs e)
        {
            if (humanCaptured)
                EndTheGame();
        }

        private void AnimateEnemy(ContentControl enemy, double from, double to, string propertToAnimate)
        {
            Storyboard storyboard = new Storyboard() { AutoReverse = true, RepeatBehavior = RepeatBehavior.Forever };
            DoubleAnimation animation = new DoubleAnimation()
            {
                From = from,
                To = to,
                Duration = new Duration(TimeSpan.FromSeconds(random.Next(4, 6))),
            };
            Storyboard.SetTarget(animation, enemy);
            Storyboard.SetTargetProperty(animation, new PropertyPath(propertToAnimate));
            storyboard.Children.Add(animation);
            storyboard.Begin();
            }

        private void Human_MouseDown(object sender, MouseButtonEventArgs e)
        {
            if (enemyTimer.IsEnabled)
            {
                humanCaptured = true;
                human.IsHitTestVisible = false;
            }
        }

        private void Target_MouseEnter(object sender, MouseEventArgs e)
        {
            if (targetTimer.IsEnabled && humanCaptured)
            {
                progressBar.Value = 0;
                Canvas.SetLeft(target, random.Next(100, (int)playArea.ActualWidth - 100));
                Canvas.SetTop(target, random.Next(100, (int)playArea.ActualHeight - 100));
                Canvas.SetLeft(human, random.Next(100, (int)playArea.ActualWidth - 100));
                Canvas.SetTop(human, random.Next(100, (int)playArea.ActualHeight - 100));
                humanCaptured = false;
                human.IsHitTestVisible = true;                  
                          
            }
        }

        private void playArea_MouseMove(object sender, MouseEventArgs e)
        {
            if (humanCaptured)
            {
                Point pointerPosition = e.GetPosition(null);
                Point relativePosition = grid.TransformToVisual(playArea).Transform(pointerPosition);
                if((Math.Abs(relativePosition.X - Canvas.GetLeft(human)) > human.ActualWidth * 3)
                    || (Math.Abs(relativePosition.Y - Canvas.GetTop(human)) > human.ActualHeight * 3))
                {
                    humanCaptured = false;
                    human.IsHitTestVisible = true;
                }
                else
                {
                    Canvas.SetLeft(human, relativePosition.X - human.ActualWidth / 2);
                    Canvas.SetTop(human, relativePosition.Y - human.ActualHeight / 2);
                }
            }

        }

        private void playArea_MouseLeave(object sender, MouseEventArgs e)
        {
            if (humanCaptured)
                EndTheGame();
        }

        
    }
}


/*XAML*/

    <Window x:Class="Save_the_Humans.MainWindow"
            xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
            xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
            xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
            xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
            xmlns:local="clr-namespace:Save_the_Humans"
            mc:Ignorable="d"
            Title="Save the Humans" Height="700" Width="1000">
        <Window.Resources>
            <ControlTemplate x:Key="EnemyTemplate" TargetType="{x:Type ContentControl}">
                <Grid>
                    <Ellipse Fill="Gray" Height="100" Stroke="Black" Width="75"/>
                    <Ellipse Fill="Black" HorizontalAlignment="Center" Height="35" Margin="40,20,70,0" Stroke="Black" VerticalAlignment="Top" Width="25" RenderTransformOrigin="0.5,0.5">
                    
                    <Ellipse.RenderTransform>
                        <TransformGroup>
                            <ScaleTransform/>
                            <SkewTransform AngleX="10"/>
                            <RotateTransform/>
                            <TranslateTransform/>
                        </TransformGroup>
                    </Ellipse.RenderTransform>
                </Ellipse>
                <Ellipse Fill="Black" HorizontalAlignment="Center" Height="35" Margin="70,20,40,0" Stroke="Black" VerticalAlignment="Top" Width="25" RenderTransformOrigin="0.5,0.5">
                    <Ellipse.RenderTransform>
                        <TransformGroup>
                            <ScaleTransform/>
                            <SkewTransform AngleX="-10"/>
                            <RotateTransform/>
                            <TranslateTransform/>
                        </TransformGroup>
                    </Ellipse.RenderTransform>
                </Ellipse>
            </Grid>
        </ControlTemplate>
    </Window.Resources>
    <Grid x:Name="grid" Background="Black">
        <Grid.ColumnDefinitions>
            <ColumnDefinition Width="140"/>
            <ColumnDefinition/>
            <ColumnDefinition Width="160"/>
        </Grid.ColumnDefinitions>
        <Grid.RowDefinitions>
            <RowDefinition/>
            <RowDefinition Height="150"/>
        </Grid.RowDefinitions>

        <Button x:Name="startButton" 
                Content="Start!" 
                HorizontalAlignment="Center" 
                Margin="0" 
                Grid.Row="1" 
                VerticalAlignment="Center" Click="StartButton_Click"/>
        <ProgressBar x:Name="progressBar"  Grid.Column="1" Grid.Row="1" Height="20"/>
        <Canvas x:Name="playArea" Height="Auto" Margin="0" Width="Auto" Grid.ColumnSpan="3" MouseLeave="playArea_MouseLeave" MouseMove="playArea_MouseMove">
            <Canvas.Background>
                <LinearGradientBrush EndPoint="0.5,1" StartPoint="0.5,0">
                    <GradientStop Color="Black" Offset="0"/>
                    <GradientStop Color="#FFFB3131" Offset="1"/>
                </LinearGradientBrush>
            </Canvas.Background>
            <StackPanel x:Name="human" Orientation="Vertical" MouseDown="Human_MouseDown">
                <Ellipse Fill="White" Height="10" Width="10"/>
                <Rectangle Fill="White" Height="25" Width="10"/>
            </StackPanel>
            <TextBlock x:Name="gameOverText" Canvas.Left="289" TextWrapping="Wrap" Text="Game Over" Canvas.Top="297" FontFamily="Arial" FontSize="72" FontWeight="Bold" FontStyle="Italic" Width="426"/>
            <Rectangle x:Name="target" Height="50" Canvas.Left="40" Canvas.Top="50" Width="50" RenderTransformOrigin="0.5,0.5" MouseEnter="Target_MouseEnter">
                <Rectangle.RenderTransform>
                    <TransformGroup>
                        <ScaleTransform/>
                        <SkewTransform/>
                        <RotateTransform Angle="45"/>
                        <TranslateTransform/>
                    </TransformGroup>
                </Rectangle.RenderTransform>
                <Rectangle.Fill>
                    <LinearGradientBrush EndPoint="0.5,1" StartPoint="0.5,0">
                        <GradientStop Color="#FF1E6DF7" Offset="0"/>
                        <GradientStop Color="White" Offset="1"/>
                    </LinearGradientBrush>
                </Rectangle.Fill>
            </Rectangle>
        </Canvas>
        <StackPanel Grid.Column="2" Orientation="Vertical" Grid.Row="1">
            <TextBlock HorizontalAlignment="Center" TextWrapping="Wrap" Text="Avoid These" VerticalAlignment="Center" FontSize="18" Foreground="White"/>
            <ContentControl HorizontalAlignment="Center" VerticalAlignment="Center" Template="{DynamicResource EnemyTemplate}">
                <Ellipse Fill="Gray" Height="100" Stroke="Black" Width="100"/>
            </ContentControl>

        </StackPanel>

    </Grid>
</Window>
