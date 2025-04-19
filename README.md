# using-microsoft-blazor-code

@page "/dashboard"

<h3>Real-time Stock Prices</h3>

<ul>
    @foreach (var stock in stocks)
    {
        <li>@stock.Symbol: @stock.Price</li>
    }
</ul>

<button class="btn btn-primary" @onclick="ToggleTheme">Toggle Theme</button>
<p>Current Theme: @themeService.CurrentTheme</p>

<h3>Counter</h3>
<p>Current count: @count</p>
<button class="btn btn-secondary" @onclick="IncrementCounter">Increment</button>

@code {
    // Theme Service
    private ThemeService themeService = new ThemeService();

    // Stock Model
    private List<Stock> stocks = new List<Stock>();

    // Counter
    private int count = 0;

    // SignalR connection for real-time updates
    protected override async Task OnInitializedAsync()
    {
        var connection = new HubConnectionBuilder()
            .WithUrl("https://your-server.com/stockHub") // Replace with actual SignalR URL
            .Build();

        connection.On<string, decimal>("ReceiveStockPrice", (symbol, price) =>
        {
            var stock = stocks.FirstOrDefault(s => s.Symbol == symbol);
            if (stock != null)
            {
                stock.Price = price;
            }
            else
            {
                stocks.Add(new Stock { Symbol = symbol, Price = price });
            }
            StateHasChanged();
        });

        await connection.StartAsync();
    }

    // Method to toggle the theme (light/dark)
    private void ToggleTheme()
    {
        themeService.ToggleTheme();
    }

    // Method to increment counter
    private void IncrementCounter()
    {
        count++;
    }

    // Stock class to hold stock data
    public class Stock
    {
        public string Symbol { get; set; }
        public decimal Price { get; set; }
    }

    // Theme Service class to manage the theme state
    public class ThemeService
    {
        private string currentTheme = "light";

        public string CurrentTheme => currentTheme;

        public void ToggleTheme()
        {
            currentTheme = currentTheme == "light" ? "dark" : "light";
        }
    }
}
