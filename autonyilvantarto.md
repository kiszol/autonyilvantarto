# Autónyilvántartó C# WPF Dokumentáció

## Áttekintés
Ez a dokumentum a C# nyelven, WPF technológiával készült **Autónyilvántartó** alkalmazás felépítését, fő funkcióit és kezelőfelületét mutatja be. A program képes CSV állományból autóadatokat beolvasni, azokat táblázatban megjeleníteni, új autókat felvenni, statisztikákat számolni, valamint oszlopdiagramot készíteni az OxyPlot segítségével.

## 1. Auto osztály (Auto.cs)
Az `Auto` osztály tartalmazza az autók tulajdonságait:

- `Rendszam`
- `Marka`
- `Tipus`
- `Fogyasztas`
- `Uzemanyag`
- `Hengerurtartalom`
- `FutottKm`

Az osztály `ToString()` metódusa a CSV formátumú kiírást segíti.

## 2. CSV kezelő osztály (CsvHandler.cs)
A `CsvHandler` osztály felelős az `autok.csv` állomány beolvasásáért és felülírásáért.

- `ReadAutos()` metódus: beolvassa a CSV-t és `Auto` objektumokat hoz létre.
- `WriteAutos()` metódus: kiírja az autók listáját CSV formátumban.
 <details>
  <summary>CsvHandler.cs</summary>

```
using System.Collections.Generic;
using System.IO;
using System.Linq;

public static class CsvHandler
{
    private const string FilePath = "autok.csv";

    public static List<Auto> ReadAutos()
    {
        List<Auto> autos = new List<Auto>();
        if (!File.Exists(FilePath)) return autos;

        var lines = File.ReadAllLines(FilePath);
        foreach (var line in lines)
        {
            var parts = line.Split(',');
            if (parts.Length == 7)
            {
                autos.Add(new Auto(
                    parts[0],
                    parts[1],
                    parts[2],
                    double.Parse(parts[3]),
                    parts[4],
                    int.Parse(parts[5]),
                    int.Parse(parts[6])
                ));
            }
        }
        return autos;
    }
}
```
</details>

## 3. MainWindow
Az alkalmazás fő ablaka. Két fő részből áll:

- **Menü** ("Fájl" menüben):
  - Új autó felvétele
  - Statisztikák
  - Diagram
- **DataGrid**: A beolvasott autók adatait táblázatban jeleníti meg (nem szerkeszthető).

Az adatok betöltése a `LoadAutos()` metódussal történik.
 <details>
  <summary>MainWindow.xaml.cs</summary>

```csharp
using System.Text;
using System.Windows;
using System.Windows.Controls;
using System.Windows.Data;
using System.Windows.Documents;
using System.Windows.Input;
using System.Windows.Media;
using System.Windows.Media.Imaging;
using System.Windows.Navigation;
using System.Windows.Shapes;
using System.Collections.Generic;
using System.IO;
using Autonyilvantarto;

namespace WpfAutoNyilvantarto
{
    public partial class MainWindow : Window
    {
            private List<Auto> autos;

            public MainWindow()
            {
                InitializeComponent();
                LoadAutos();
            }

            private void LoadAutos()
            {
                autos = CsvHandler.ReadAutos();
                AutosDataGrid.ItemsSource = autos;
            }

            private void AddAuto_Click(object sender, RoutedEventArgs e)
            {
                var addWindow = new AddAutoWindow();
                if (addWindow.ShowDialog() == true)
                {
                    autos.Add(addWindow.NewAuto);
                    CsvHandler.WriteAutos(autos);
                    LoadAutos();
                }
            }

            private void ShowStatistics_Click(object sender, RoutedEventArgs e)
            {
                new StatisticsWindow(autos).ShowDialog();
            }

            private void ShowDiagram_Click(object sender, RoutedEventArgs e)
            {
                new DiagramWindow(autos).ShowDialog();
            }
        }

}
```
</details>
<details>
  <summary>MainWindow.xaml</summary>

```xaml
<Window x:Class="Autonyilvantarto.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="Autónyilvántartó 1.0" Height="450" Width="800">
    <Grid>
        <Grid.RowDefinitions>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="*"/>
        </Grid.RowDefinitions>

        <Menu Grid.Row="0">
            <MenuItem Header="Fájl">
                <MenuItem Header="Új autó felvétele" Click="AddAuto_Click"/>
                <MenuItem Header="Statisztikák" Click="ShowStatistics_Click"/>
                <MenuItem Header="Diagram" Click="ShowDiagram_Click"/>
            </MenuItem>
        </Menu>

        <DataGrid Grid.Row="1" x:Name="AutosDataGrid" AutoGenerateColumns="False" IsReadOnly="True" Margin="10">
            <DataGrid.Columns>
                <DataGridTextColumn Header="Rendszám" Binding="{Binding Rendszam}"/>
                <DataGridTextColumn Header="Márka" Binding="{Binding Marka}"/>
                <DataGridTextColumn Header="Típus" Binding="{Binding Tipus}"/>
                <DataGridTextColumn Header="Fogyasztás (l/100km)" Binding="{Binding Fogyasztas}"/>
                <DataGridTextColumn Header="Üzemanyag" Binding="{Binding Uzemanyag}"/>
                <DataGridTextColumn Header="Hengerűrtartalom" Binding="{Binding Hengerurtartalom}"/>
                <DataGridTextColumn Header="Futott km" Binding="{Binding FutottKm}"/>
            </DataGrid.Columns>
        </DataGrid>
    </Grid>
</Window>
```
</details>

## 4. AddAutoWindow
Ez az ablak lehetőséget ad új autó felvételére:

- 7 beviteli mező:
  - Rendszám
  - Márka
  - Típus
  - Fogyasztás
  - Üzemanyag (ComboBox)
  - Hengerűrtartalom
  - Futott km

- **„Mentés”** gomb:
  - Érvényesítés után új `Auto` objektum létrehozása
  - Adatok mentése a CSV-be
  - Ablak bezárása


<details>
  <summary>AddAutoWindow.xaml.cs</summary>

```csharp
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
using System.Windows.Shapes;
using WpfAutoNyilvantarto;

namespace Autonyilvantarto
{
    public partial class AddAutoWindow : Window
    {
        public Auto NewAuto { get; private set; }

        public AddAutoWindow()
        {
            InitializeComponent();
        }

        private void Save_Click(object sender, RoutedEventArgs e)
        {
            try
            {
                NewAuto = new Auto(
                    RendszamTextBox.Text,
                    MarkaTextBox.Text,
                    TipusTextBox.Text,
                    double.Parse(FogyasztasTextBox.Text),
                    (UzemanyagComboBox.SelectedItem as ComboBoxItem)?.Content.ToString(),
                    int.Parse(HengerurtartalomTextBox.Text),
                    int.Parse(FutottKmTextBox.Text)
                );
                DialogResult = true;
                Close();
            }
            catch
            {
                MessageBox.Show("Érvénytelen adatok, kérlek ellenőrizd!");
            }
        }
    }
}
```
</details>

<details>
  <summary>AddAutoWindow.xaml</summary>

```xaml
<Window x:Class="Autonyilvantarto.AddAutoWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="Új autó felvétele" Height="400" Width="400">
    <Grid Margin="10">
        <Grid.RowDefinitions>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="*"/>
            <RowDefinition Height="Auto"/>
        </Grid.RowDefinitions>

        <Label Content="Rendszám:" Grid.Row="0"/>
        <TextBox x:Name="RendszamTextBox" Grid.Row="1" Margin="0,0,0,10"/>

        <Label Content="Márka:" Grid.Row="2"/>
        <TextBox x:Name="MarkaTextBox" Grid.Row="3" Margin="0,0,0,10"/>

        <Label Content="Típus:" Grid.Row="4"/>
        <TextBox x:Name="TipusTextBox" Grid.Row="5" Margin="0,0,0,10"/>

        <Label Content="Fogyasztás:" Grid.Row="6"/>
        <TextBox x:Name="FogyasztasTextBox" Grid.Row="7" Margin="0,0,0,10"/>

        <Label Content="Üzemanyag:" Grid.Row="8"/>
        <ComboBox x:Name="UzemanyagComboBox" Grid.Row="9" Margin="0,0,0,10">
            <ComboBoxItem Content="Benzin"/>
            <ComboBoxItem Content="Dízel"/>
        </ComboBox>

        <Label Content="Hengerűrtartalom:" Grid.Row="10"/>
        <TextBox x:Name="HengerurtartalomTextBox" Grid.Row="11" Margin="0,0,0,10"/>

        <Label Content="Futott km:" Grid.Row="12"/>
        <TextBox x:Name="FutottKmTextBox" Grid.Row="13" Margin="0,0,0,10"/>

        <Button Content="Mentés" Grid.Row="14" Click="Save_Click" Width="100" HorizontalAlignment="Right"/>
    </Grid>
</Window>
```
</details>


## 5. StatisticsWindow
Statisztikai áttekintést ad az autók adatairól.

- A márka ComboBox alapértelmezetten **„BMW”**-re van állítva
- Lekérhető statisztikák:
  - Átlagfogyasztás adott márkára
  - Átlag hengerűrtartalom adott márkára
  - Benzinesek átlagfogyasztása adott márkán belül
  - Dízelek összes futott kilométere

A ComboBox változtatásával a statisztikák automatikusan frissülnek.
<details>
  <summary>StatisticsWindow.xaml.cs</summary>

```csharp
using System.Collections.Generic;
using System.Linq;
using System.Windows;
using System.Windows.Controls;
using WpfAutoNyilvantarto;

namespace Autonyilvantarto
{
    public partial class StatisticsWindow : Window
    {
        private List<Auto> autos;

        public StatisticsWindow(List<Auto> autos)
        {
            InitializeComponent();
            this.autos = autos;

            var markak = autos.Select(a => a.Marka).Distinct().ToList();
            MarkaComboBox.ItemsSource = markak;
            MarkaComboBox.SelectedIndex = markak.IndexOf("BMW"); 
            UpdateStatistics("BMW");
        }

        private void MarkaComboBox_SelectionChanged(object sender, SelectionChangedEventArgs e)
        {
            if (MarkaComboBox.SelectedItem != null)
            {
                UpdateStatistics(MarkaComboBox.SelectedItem.ToString());
            }
        }

        private void UpdateStatistics(string marka)
        {
            var markaAutos = autos.Where(a => a.Marka == marka).ToList();

            if (markaAutos.Any())
            {
                double avgFogyasztas = markaAutos.Average(a => a.Fogyasztas);
                AvgFogyasztasLabel.Content = $"Átlagfogyasztás: {avgFogyasztas:F2} l/100km";

                double avgHengerurtartalom = markaAutos.Average(a => a.Hengerurtartalom);
                AvgHengerurtartalomLabel.Content = $"Átlag hengerűrtartalom: {avgHengerurtartalom:F2} cm³";

                var benzinesAutos = markaAutos.Where(a => a.Uzemanyag == "Benzin").ToList();
                double benzinesAvgFogyasztas = benzinesAutos.Any() ? benzinesAutos.Average(a => a.Fogyasztas) : 0;
                BenzinAvgFogyasztasLabel.Content = $"Benzinesek átlagfogyasztása: {benzinesAvgFogyasztas:F2} l/100km";

                var dieselAutos = markaAutos.Where(a => a.Uzemanyag == "Dízel").ToList();
                int dieselTotalKm = dieselAutos.Sum(a => a.FutottKm);
                DieselTotalKmLabel.Content = $"Dízelek össz km: {dieselTotalKm} km";
            }
            else
            {
                AvgFogyasztasLabel.Content = "Átlagfogyasztás: N/A";
                AvgHengerurtartalomLabel.Content = "Átlag hengerűrtartalom: N/A";
                BenzinAvgFogyasztasLabel.Content = "Benzinesek átlagfogyasztása: N/A";
                DieselTotalKmLabel.Content = "Dízelek össz km: N/A";
            }
        }
    }
}
```
</details>
<details>
  <summary>StatisticsWindow.xaml</summary>

```xaml
<Window x:Class="Autonyilvantarto.StatisticsWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="Statisztikák" Height="300" Width="400">
    <Grid Margin="10">
        <Grid.RowDefinitions>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="Auto"/>
        </Grid.RowDefinitions>

        <Label Content="Márka:" Grid.Row="0"/>
        <ComboBox x:Name="MarkaComboBox" Grid.Row="1" Margin="0,0,0,10" SelectionChanged="MarkaComboBox_SelectionChanged"/>

        <Label x:Name="AvgFogyasztasLabel" Content="Átlagfogyasztás: " Grid.Row="2"/>
        <Label x:Name="AvgHengerurtartalomLabel" Content="Átlag hengerűrtartalom: " Grid.Row="3"/>
        <Label x:Name="BenzinAvgFogyasztasLabel" Content="Benzinesek átlagfogyasztása: " Grid.Row="4"/>
        <Label x:Name="DieselTotalKmLabel" Content="Dízelek össz km: " Grid.Row="4" Margin="0,31,0,-31"/>
    </Grid>
</Window>
```
</details>

## 6. DiagramWindow
Ablak, amely **OxyPlot** segítségével oszlopdiagramot rajzol az autók fogyasztásáról.

- **X tengely**: autók (márka + típus)
- **Y tengely**: fogyasztás 
- Az oszlopdiagram vizualizálja az összes autó fogyasztását
