# CoinMarketCap API Client for Laravel & PHP

![CoinMarketCap PHP SDK](https://github.com/user-attachments/assets/23b58c63-2624-4689-9f21-2566184e4608)

[![Latest Version on Packagist](https://img.shields.io/packagist/v/tigusigalpa/coinmarketcap.svg?style=flat-square)](https://packagist.org/packages/tigusigalpa/coinmarketcap)
[![License](https://img.shields.io/packagist/l/tigusigalpa/coinmarketcap.svg?style=flat-square)](https://packagist.org/packages/tigusigalpa/coinmarketcap)

PHP client for the [CoinMarketCap API v1](https://coinmarketcap.com/api/documentation/v1/). Works standalone or with Laravel (service provider, facade, DI). Covers cryptocurrency listings, quotes, OHLCV, exchanges, global metrics, and price conversion.

Built on PHP 8.1+ with strict types, PSR-18 HTTP client, PHPStan level 8, and PHPUnit tests.

## Features

### Core

- PHP 8.1+ with strict types, readonly properties, enums
- PSR-18 HTTP client abstraction
- Fluent `ClientBuilder` for configuration
- Laravel service provider, facade, dependency injection
- Type-safe DTO models for all responses
- Custom exceptions for rate limits, auth errors, invalid requests
- PHPUnit test suite, PHPStan level 8, PSR-12 code style

### API coverage

- Real-time prices and market cap rankings
- Historical quotes and OHLCV data
- Price conversion (crypto ↔ fiat)
- Trending coins, gainers, losers
- Cryptocurrency metadata (logos, links, descriptions)
- Exchange listings, trading pairs, market pairs
- Global market metrics (total market cap, BTC dominance)

### Developer experience

- Composer install, works out of the box with Laravel
- Sandbox mode for testing without production credits
- Rate limit detection and handling
- Configurable via `.env` and config files

## Requirements

- PHP 8.1 or higher
- Laravel 10.x or 11.x
- CoinMarketCap API key ([Get one here](https://coinmarketcap.com/api/))

## Installation

### 1. Install via Composer

```bash
composer require tigusigalpa/coinmarketcap
```

Laravel auto-discovery registers the service provider automatically.

### 2. Get your API key

1. Visit [CoinMarketCap API Portal](https://coinmarketcap.com/api/)
2. Sign up for an account (Basic plan: 10,000 calls/month, free)
3. Copy your API key from the dashboard

### 3. Publish config (Laravel)

```bash
php artisan vendor:publish --tag=coinmarketcap-config
```

### 4. Configure `.env`

```env
# CoinMarketCap API Configuration
COINMARKETCAP_API_KEY=your-api-key-here
COINMARKETCAP_BASE_URL=https://pro-api.coinmarketcap.com
COINMARKETCAP_TIMEOUT=30
COINMARKETCAP_USE_SANDBOX=false
```

### 5. Clear config cache

```bash
php artisan config:clear
```

### Standalone PHP (without Laravel)

```php
use Tigusigalpa\CoinMarketCap\ClientBuilder;

$client = (new ClientBuilder())
    ->setApiKey('your-coinmarketcap-api-key')
    ->build();
```

### Verify installation

```php
use Tigusigalpa\CoinMarketCap\Facades\CoinMarketCap;

// Get Bitcoin price
$bitcoin = CoinMarketCap::cryptocurrency()->quotesLatest([
    'symbol' => 'BTC',
    'convert' => 'USD'
]);

echo "Bitcoin Price: $" . $bitcoin['data']['BTC']['quote']['USD']['price'];
```

## Quick Start

### Using Facade (Recommended for Laravel)

```php
use Tigusigalpa\CoinMarketCap\Facades\CoinMarketCap;

// Get top 10 cryptocurrencies
$listings = CoinMarketCap::cryptocurrency()->listingsLatest([
    'start' => 1,
    'limit' => 10,
    'convert' => 'USD'
]);

// Get Bitcoin and Ethereum quotes
$quotes = CoinMarketCap::cryptocurrency()->quotesLatest([
    'symbol' => 'BTC,ETH',
    'convert' => 'USD'
]);
```

### Using Dependency Injection

```php
use Tigusigalpa\CoinMarketCap\Client;

class CryptoController extends Controller
{
    public function __construct(private Client $client)
    {
    }

    public function index()
    {
        $listings = $this->client->cryptocurrency()->listingsLatest([
            'limit' => 10
        ]);
        
        return view('crypto.index', compact('listings'));
    }
}
```

### Standalone Usage (Outside Laravel)

```php
use Tigusigalpa\CoinMarketCap\ClientBuilder;

$client = (new ClientBuilder())
    ->setApiKey('your-api-key')
    ->setBaseUrl('https://pro-api.coinmarketcap.com')
    ->setTimeout(30)
    ->build();

$quote = $client->cryptocurrency()->quotesLatest([
    'symbol' => 'BTC,ETH',
    'convert' => 'USD'
]);
```

## Available API Methods

### Cryptocurrency API

| Method                    | Endpoint                                     | Description                                  |
|---------------------------|----------------------------------------------|----------------------------------------------|
| `listingsLatest()`        | `/v1/cryptocurrency/listings/latest`         | Get list of cryptocurrencies by market cap   |
| `listingsHistorical()`    | `/v1/cryptocurrency/listings/historical`     | Historical listings                          |
| `quotesLatest()`          | `/v2/cryptocurrency/quotes/latest`           | Current quotes for specific cryptocurrencies |
| `quotesHistorical()`      | `/v2/cryptocurrency/quotes/historical`       | Historical quotes                            |
| `info()`                  | `/v2/cryptocurrency/info`                    | Metadata (logos, links, description)         |
| `map()`                   | `/v1/cryptocurrency/map`                     | CoinMarketCap ID mapping                     |
| `ohlcvLatest()`           | `/v2/cryptocurrency/ohlcv/latest`            | Latest OHLCV data                            |
| `ohlcvHistorical()`       | `/v2/cryptocurrency/ohlcv/historical`        | Historical OHLCV data                        |
| `categories()`            | `/v1/cryptocurrency/categories`              | Cryptocurrency categories                    |
| `trendingLatest()`        | `/v1/cryptocurrency/trending/latest`         | Trending cryptocurrencies                    |
| `trendingGainersLosers()` | `/v1/cryptocurrency/trending/gainers-losers` | Top gainers and losers                       |

### Exchange API

| Method                | Endpoint                           | Description                         |
|-----------------------|------------------------------------|-------------------------------------|
| `listingsLatest()`    | `/v1/exchange/listings/latest`     | List of exchanges by trading volume |
| `quotesLatest()`      | `/v1/exchange/quotes/latest`       | Current exchange data               |
| `quotesHistorical()`  | `/v1/exchange/quotes/historical`   | Historical data                     |
| `info()`              | `/v1/exchange/info`                | Exchange metadata                   |
| `map()`               | `/v1/exchange/map`                 | Exchange ID mapping                 |
| `marketPairsLatest()` | `/v1/exchange/market-pairs/latest` | Exchange trading pairs              |

### Global Metrics API

| Method               | Endpoint                               | Description               |
|----------------------|----------------------------------------|---------------------------|
| `quotesLatest()`     | `/v1/global-metrics/quotes/latest`     | Current global metrics    |
| `quotesHistorical()` | `/v1/global-metrics/quotes/historical` | Historical global metrics |

### Tools API

| Method              | Endpoint                     | Description                       |
|---------------------|------------------------------|-----------------------------------|
| `priceConversion()` | `/v2/tools/price-conversion` | Convert prices between currencies |

## Usage examples

### Portfolio tracker

```php
use Tigusigalpa\CoinMarketCap\Facades\CoinMarketCap;

class CryptoPortfolioController extends Controller
{
    public function showPortfolio()
    {
        // User's cryptocurrency holdings
        $portfolio = [
            'BTC' => 0.5,      // 0.5 Bitcoin
            'ETH' => 10,       // 10 Ethereum
            'BNB' => 50,       // 50 Binance Coin
            'ADA' => 1000,     // 1000 Cardano
        ];
        
        // Get current prices for all holdings
        $quotes = CoinMarketCap::cryptocurrency()->quotesLatest([
            'symbol' => implode(',', array_keys($portfolio)),
            'convert' => 'USD'
        ]);
        
        $totalValue = 0;
        $holdings = [];
        
        foreach ($portfolio as $symbol => $amount) {
            $price = $quotes['data'][$symbol]['quote']['USD']['price'];
            $value = $amount * $price;
            $totalValue += $value;
            
            $holdings[] = [
                'name' => $quotes['data'][$symbol]['name'],
                'symbol' => $symbol,
                'amount' => $amount,
                'price' => $price,
                'value' => $value,
                'change_24h' => $quotes['data'][$symbol]['quote']['USD']['percent_change_24h']
            ];
        }
        
        return view('portfolio', compact('holdings', 'totalValue'));
    }
}
```

### Price alert system

```php
use Tigusigalpa\CoinMarketCap\Facades\CoinMarketCap;
use Illuminate\Support\Facades\Notification;

class CryptoPriceAlertService
{
    public function checkPriceAlerts()
    {
        // Get user-defined price alerts from database
        $alerts = PriceAlert::where('active', true)->get();
        
        // Group alerts by cryptocurrency symbol
        $symbols = $alerts->pluck('symbol')->unique()->implode(',');
        
        // Fetch current prices
        $quotes = CoinMarketCap::cryptocurrency()->quotesLatest([
            'symbol' => $symbols,
            'convert' => 'USD'
        ]);
        
        foreach ($alerts as $alert) {
            $currentPrice = $quotes['data'][$alert->symbol]['quote']['USD']['price'];
            
            // Check if price crossed threshold
            if ($alert->type === 'above' && $currentPrice >= $alert->target_price) {
                $this->sendAlert($alert, $currentPrice);
            } elseif ($alert->type === 'below' && $currentPrice <= $alert->target_price) {
                $this->sendAlert($alert, $currentPrice);
            }
        }
    }
    
    private function sendAlert($alert, $currentPrice)
    {
        Notification::send($alert->user, new CryptoPriceAlert([
            'symbol' => $alert->symbol,
            'current_price' => $currentPrice,
            'target_price' => $alert->target_price,
        ]));
        
        $alert->update(['active' => false]);
    }
}
```

### Top cryptocurrencies by market cap

```php
use Tigusigalpa\CoinMarketCap\Facades\CoinMarketCap;

class CryptoMarketController extends Controller
{
    public function topCryptocurrencies()
    {
        $top100 = CoinMarketCap::cryptocurrency()->listingsLatest([
            'start' => 1,
            'limit' => 100,
            'convert' => 'USD',
            'sort' => 'market_cap',
            'sort_dir' => 'desc'
        ]);
        
        $cryptocurrencies = collect($top100['data'])->map(function ($crypto) {
            return [
                'rank' => $crypto['cmc_rank'],
                'name' => $crypto['name'],
                'symbol' => $crypto['symbol'],
                'price' => $crypto['quote']['USD']['price'],
                'market_cap' => $crypto['quote']['USD']['market_cap'],
                'volume_24h' => $crypto['quote']['USD']['volume_24h'],
                'change_1h' => $crypto['quote']['USD']['percent_change_1h'],
                'change_24h' => $crypto['quote']['USD']['percent_change_24h'],
                'change_7d' => $crypto['quote']['USD']['percent_change_7d'],
            ];
        });
        
        return view('crypto.market', compact('cryptocurrencies'));
    }
}
```

### Price converter

```php
use Tigusigalpa\CoinMarketCap\Facades\CoinMarketCap;

class CryptoConverterController extends Controller
{
    public function convert(Request $request)
    {
        $conversion = CoinMarketCap::tools()->priceConversion([
            'amount' => $request->amount,
            'symbol' => $request->from_currency,
            'convert' => $request->to_currency
        ]);
        
        $result = [
            'from' => [
                'amount' => $request->amount,
                'currency' => $request->from_currency,
            ],
            'to' => [
                'amount' => $conversion['data']['quote'][$request->to_currency]['price'],
                'currency' => $request->to_currency,
            ],
            'rate' => $conversion['data']['quote'][$request->to_currency]['price'] / $request->amount,
            'timestamp' => $conversion['data']['last_updated']
        ];
        
        return response()->json($result);
    }
}
```

### Trending cryptocurrencies

```php
use Tigusigalpa\CoinMarketCap\Facades\CoinMarketCap;

class TrendingCryptoController extends Controller
{
    public function trending()
    {
        // Get trending cryptocurrencies
        $trending = CoinMarketCap::cryptocurrency()->trendingLatest([
            'limit' => 10,
            'convert' => 'USD'
        ]);
        
        // Get top gainers and losers
        $gainersLosers = CoinMarketCap::cryptocurrency()->trendingGainersLosers([
            'limit' => 10,
            'convert' => 'USD',
            'time_period' => '24h'
        ]);
        
        return view('crypto.trending', [
            'trending' => $trending['data'],
            'gainers' => $gainersLosers['data']['gainers'],
            'losers' => $gainersLosers['data']['losers']
        ]);
    }
}
```

### Cryptocurrency metadata

```php
use Tigusigalpa\CoinMarketCap\Facades\CoinMarketCap;

$info = CoinMarketCap::cryptocurrency()->info([
    'symbol' => 'BTC,ETH,BNB',
]);

foreach ($info['data'] as $symbol => $crypto) {
    echo "Name: {$crypto['name']}\n";
    echo "Symbol: {$crypto['symbol']}\n";
    echo "Description: {$crypto['description']}\n";
    echo "Website: {$crypto['urls']['website'][0]}\n";
    echo "Logo: {$crypto['logo']}\n";
    echo "Twitter: {$crypto['urls']['twitter'][0]}\n";
}
```

### Global market metrics

```php
use Tigusigalpa\CoinMarketCap\Facades\CoinMarketCap;

$metrics = CoinMarketCap::globalMetrics()->quotesLatest([
    'convert' => 'USD'
]);

$data = $metrics['data'];

$marketStats = [
    'total_market_cap' => $data['quote']['USD']['total_market_cap'],
    'total_volume_24h' => $data['quote']['USD']['total_volume_24h'],
    'btc_dominance' => $data['btc_dominance'],
    'eth_dominance' => $data['eth_dominance'],
    'active_cryptocurrencies' => $data['active_cryptocurrencies'],
    'active_exchanges' => $data['active_exchanges'],
    'active_market_pairs' => $data['active_market_pairs'],
];

return view('crypto.global-stats', compact('marketStats'));
```

### Exchange comparison

```php
use Tigusigalpa\CoinMarketCap\Facades\CoinMarketCap;

class ExchangeComparisonController extends Controller
{
    public function compareExchanges()
    {
        $exchanges = CoinMarketCap::exchange()->listingsLatest([
            'limit' => 50,
            'sort' => 'volume_24h',
            'convert' => 'USD'
        ]);
        
        $exchangeData = collect($exchanges['data'])->map(function ($exchange) {
            return [
                'name' => $exchange['name'],
                'volume_24h' => $exchange['quote']['USD']['volume_24h'],
                'num_market_pairs' => $exchange['num_market_pairs'],
                'traffic_score' => $exchange['traffic_score'] ?? 0,
            ];
        });
        
        return view('crypto.exchanges', compact('exchangeData'));
    }
}
```

### Historical price data

```php
use Tigusigalpa\CoinMarketCap\Facades\CoinMarketCap;

class HistoricalDataController extends Controller
{
    public function getHistoricalPrices($symbol)
    {
        $historical = CoinMarketCap::cryptocurrency()->quotesHistorical([
            'symbol' => $symbol,
            'convert' => 'USD',
            'time_start' => now()->subDays(30)->timestamp,
            'time_end' => now()->timestamp,
            'interval' => 'daily'
        ]);
        
        $priceData = collect($historical['data']['quotes'])->map(function ($quote) {
            return [
                'date' => $quote['timestamp'],
                'price' => $quote['quote']['USD']['price'],
                'volume' => $quote['quote']['USD']['volume_24h'],
                'market_cap' => $quote['quote']['USD']['market_cap'],
            ];
        });
        
        return response()->json($priceData);
    }
}
```

### Caching

```php
use Tigusigalpa\CoinMarketCap\Facades\CoinMarketCap;
use Illuminate\Support\Facades\Cache;

class CachedCryptoService
{
    public function getTopCryptocurrencies($limit = 100)
    {
        $cacheKey = "crypto_top_{$limit}";
        
        return Cache::remember($cacheKey, now()->addMinutes(5), function () use ($limit) {
            return CoinMarketCap::cryptocurrency()->listingsLatest([
                'limit' => $limit,
                'convert' => 'USD'
            ]);
        });
    }
    
    public function getBitcoinPrice()
    {
        return Cache::remember('btc_price', now()->addMinutes(1), function () {
            $quote = CoinMarketCap::cryptocurrency()->quotesLatest([
                'symbol' => 'BTC',
                'convert' => 'USD'
            ]);
            
            return $quote['data']['BTC']['quote']['USD']['price'];
        });
    }
}

## Error handling

### Exception types

| Exception | HTTP Code | Description | Common Causes |
|-----------|-----------|-------------|---------------|
| `AuthenticationException` | 401 | Invalid or missing API key | Wrong API key, expired key |
| `RateLimitException` | 429 | API rate limit exceeded | Too many requests, plan limit reached |
| `InvalidRequestException` | 400 | Invalid request parameters | Wrong parameter format, missing required fields |
| `NotFoundException` | 404 | Resource not found | Invalid cryptocurrency ID or symbol |
| `ApiException` | Various | General API error | Network issues, server errors |

### Basic Error Handling

```php
use Tigusigalpa\CoinMarketCap\Facades\CoinMarketCap;
use Tigusigalpa\CoinMarketCap\Exception\RateLimitException;
use Tigusigalpa\CoinMarketCap\Exception\AuthenticationException;
use Tigusigalpa\CoinMarketCap\Exception\InvalidRequestException;
use Tigusigalpa\CoinMarketCap\Exception\NotFoundException;
use Tigusigalpa\CoinMarketCap\Exception\ApiException;
use Illuminate\Support\Facades\Log;

try {
    $listings = CoinMarketCap::cryptocurrency()->listingsLatest([
        'limit' => 100,
        'convert' => 'USD'
    ]);
    
    // Process cryptocurrency data
    
} catch (AuthenticationException $e) {
    // Invalid API key - check your .env configuration
    Log::error('CoinMarketCap Authentication Error', [
        'message' => $e->getMessage(),
        'status_code' => $e->statusCode,
    ]);
    
    return response()->json([
        'error' => 'API authentication failed. Please check your API key.'
    ], 401);
    
} catch (RateLimitException $e) {
    // Rate limit exceeded - implement retry logic
    $retryAfter = $e->getRetryAfter();
    
    Log::warning('CoinMarketCap Rate Limit Exceeded', [
        'message' => $e->getMessage(),
        'retry_after' => $retryAfter,
    ]);
    
    return response()->json([
        'error' => 'Rate limit exceeded. Please try again later.',
        'retry_after' => $retryAfter
    ], 429);
    
} catch (InvalidRequestException $e) {
    // Invalid parameters - check your request
    Log::error('CoinMarketCap Invalid Request', [
        'message' => $e->getMessage(),
        'response' => $e->getResponse(),
    ]);
    
    return response()->json([
        'error' => 'Invalid request parameters.',
        'details' => $e->getMessage()
    ], 400);
    
} catch (NotFoundException $e) {
    // Resource not found - invalid cryptocurrency ID/symbol
    Log::warning('CoinMarketCap Resource Not Found', [
        'message' => $e->getMessage(),
    ]);
    
    return response()->json([
        'error' => 'Cryptocurrency not found.'
    ], 404);
    
} catch (ApiException $e) {
    // General API error
    Log::error('CoinMarketCap API Error', [
        'message' => $e->getMessage(),
        'status_code' => $e->statusCode,
        'response' => $e->getResponse(),
    ]);
    
    return response()->json([
        'error' => 'An error occurred while fetching cryptocurrency data.'
    ], 500);
}
```

### Retry logic

```php
use Tigusigalpa\CoinMarketCap\Facades\CoinMarketCap;
use Tigusigalpa\CoinMarketCap\Exception\RateLimitException;

class ResilientCryptoService
{
    public function getCryptoPricesWithRetry($symbols, $maxRetries = 3)
    {
        $attempt = 0;
        
        while ($attempt < $maxRetries) {
            try {
                return CoinMarketCap::cryptocurrency()->quotesLatest([
                    'symbol' => $symbols,
                    'convert' => 'USD'
                ]);
                
            } catch (RateLimitException $e) {
                $attempt++;
                
                if ($attempt >= $maxRetries) {
                    throw $e;
                }
                
                $retryAfter = $e->getRetryAfter() ?? 60;
                Log::info("Rate limit hit. Retrying after {$retryAfter} seconds...");
                sleep($retryAfter);
            }
        }
    }
}
```

### Fallback with cache

```php
use Tigusigalpa\CoinMarketCap\Facades\CoinMarketCap;
use Tigusigalpa\CoinMarketCap\Exception\ApiException;
use Illuminate\Support\Facades\Cache;

class CryptoServiceWithFallback
{
    public function getBitcoinPrice()
    {
        try {
            $quote = CoinMarketCap::cryptocurrency()->quotesLatest([
                'symbol' => 'BTC',
                'convert' => 'USD'
            ]);
            
            $price = $quote['data']['BTC']['quote']['USD']['price'];
            
            // Cache successful response
            Cache::put('btc_price_fallback', $price, now()->addHours(1));
            
            return $price;
            
        } catch (ApiException $e) {
            Log::error('Failed to fetch Bitcoin price', ['error' => $e->getMessage()]);
            
            // Return cached data as fallback
            return Cache::get('btc_price_fallback', 0);
        }
    }
}
```

### Monitoring API usage

```php
use Tigusigalpa\CoinMarketCap\Facades\CoinMarketCap;

class ApiUsageMonitor
{
    public function trackApiUsage()
    {
        try {
            $response = CoinMarketCap::cryptocurrency()->listingsLatest(['limit' => 1]);
            
            $creditCount = $response['status']['credit_count'];
            $timestamp = $response['status']['timestamp'];
            
            // Log API usage
            Log::info('CoinMarketCap API Usage', [
                'credits_used' => $creditCount,
                'timestamp' => $timestamp,
            ]);
            
            // Store in database for monitoring
            ApiUsage::create([
                'credits_used' => $creditCount,
                'endpoint' => 'cryptocurrency/listings/latest',
                'called_at' => $timestamp,
            ]);
            
        } catch (\Exception $e) {
            Log::error('Failed to track API usage', ['error' => $e->getMessage()]);
        }
    }
}
```

### Common Issues and Solutions

**Issue: "Authentication failed" error**

- **Solution**: Verify your API key in `.env` file is correct
- Check that `COINMARKETCAP_API_KEY` is set properly
- Ensure no extra spaces or quotes around the API key
- Run `php artisan config:clear` after changing `.env`

**Issue: Rate limit exceeded**

- **Solution**: Implement caching to reduce API calls
- Upgrade your CoinMarketCap API plan for higher limits
- Use the sandbox environment for testing
- Implement exponential backoff retry logic

**Issue: Invalid cryptocurrency symbol**

- **Solution**: Use the `map()` endpoint to get valid symbols
- Check CoinMarketCap documentation for correct symbol format
- Use cryptocurrency ID instead of symbol for more reliability

**Issue: Timeout errors**

- **Solution**: Increase timeout in configuration
- Check your network connection
- Use asynchronous requests for multiple API calls

## Sandbox mode

### Via `.env`

```env
COINMARKETCAP_USE_SANDBOX=true
COINMARKETCAP_API_KEY=your-sandbox-api-key
```

### Enable Sandbox Programmatically

```php
use Tigusigalpa\CoinMarketCap\ClientBuilder;

$client = (new ClientBuilder())
    ->setApiKey('your-sandbox-api-key')
    ->useSandbox()
    ->build();

// Now all API calls will use the sandbox environment
$quotes = $client->cryptocurrency()->quotesLatest([
    'symbol' => 'BTC',
    'convert' => 'USD'
]);
```

Sandbox uses the same response format as production, so you can test without spending credits.

## Performance tips

### Caching

```php
use Illuminate\Support\Facades\Cache;
use Tigusigalpa\CoinMarketCap\Facades\CoinMarketCap;

// Cache for 5 minutes
$cryptoData = Cache::remember('crypto_top_100', 300, function () {
    return CoinMarketCap::cryptocurrency()->listingsLatest([
        'limit' => 100,
        'convert' => 'USD'
    ]);
});
```

### Batch requests

```php
// Good: Single API call for multiple cryptocurrencies
$quotes = CoinMarketCap::cryptocurrency()->quotesLatest([
    'symbol' => 'BTC,ETH,BNB,ADA,XRP,SOL,DOT,DOGE',
    'convert' => 'USD'
]);

// Bad: Multiple API calls
// Don't do this - wastes API credits
foreach (['BTC', 'ETH', 'BNB'] as $symbol) {
    $quote = CoinMarketCap::cryptocurrency()->quotesLatest([
        'symbol' => $symbol
    ]);
}
```

### Background jobs

```php
use Illuminate\Bus\Queueable;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Contracts\Queue\ShouldQueue;

class UpdateCryptoPricesJob implements ShouldQueue
{
    use Queueable, InteractsWithQueue;
    
    public function handle()
    {
        $prices = CoinMarketCap::cryptocurrency()->listingsLatest([
            'limit' => 100
        ]);
        
        // Update database
        foreach ($prices['data'] as $crypto) {
            CryptoCurrency::updateOrCreate(
                ['symbol' => $crypto['symbol']],
                ['price' => $crypto['quote']['USD']['price']]
            );
        }
    }
}

// Dispatch job
UpdateCryptoPricesJob::dispatch();
```

### Store historical data locally

```php
// Fetch once and store
$historical = CoinMarketCap::cryptocurrency()->quotesHistorical([
    'symbol' => 'BTC',
    'time_start' => now()->subDays(30)->timestamp,
    'interval' => 'daily'
]);

foreach ($historical['data']['quotes'] as $quote) {
    HistoricalPrice::create([
        'symbol' => 'BTC',
        'price' => $quote['quote']['USD']['price'],
        'date' => $quote['timestamp']
    ]);
}

// Query from database for charts
$chartData = HistoricalPrice::where('symbol', 'BTC')
    ->whereBetween('date', [now()->subDays(30), now()])
    ->get();
```

### Request only needed fields

```php
// Only fetch required fields
$listings = CoinMarketCap::cryptocurrency()->listingsLatest([
    'limit' => 10,
    'convert' => 'USD',
    'aux' => 'num_market_pairs,cmc_rank' // Only specific auxiliary fields
]);
```

## FAQ

### General Questions

**Q: Is this package free to use?**
A: The package is open-source (MIT). You need a CoinMarketCap API key — the Basic plan is free (10,000 calls/month).

**Q: What's the difference between this package and calling the API directly?**
A: Type-safe models, automatic error handling, Laravel integration, retry logic, and a clean interface. Saves boilerplate.

**Q: Can I use this package outside of Laravel?**
A: Yes. Use the standalone `ClientBuilder` in any PHP 8.1+ project.

**Q: Does this support all CoinMarketCap API endpoints?**
A: All major cryptocurrency, exchange, global metrics, and tools endpoints are covered.

### API & Authentication

**Q: How do I get a CoinMarketCap API key?**
A: Visit [coinmarketcap.com/api](https://coinmarketcap.com/api/), sign up for a free account, and get your API key from
the dashboard.

**Q: What are the API rate limits?**
A: The free Basic plan includes 10,000 calls/month. Paid plans offer higher limits. The package automatically handles
rate limit errors.

**Q: Can I use multiple API keys?**
A: Yes, you can create multiple client instances with different API keys using the ClientBuilder.

**Q: Is my API key secure?**
A: Yes, store your API key in the `.env` file (never commit it to version control). The package sends it securely via
HTTPS headers.

### Cryptocurrency Data

**Q: How often is cryptocurrency data updated?**
A: CoinMarketCap updates prices every 60 seconds for most cryptocurrencies. Implement caching to avoid unnecessary API
calls.

**Q: Can I get historical cryptocurrency prices?**
A: Yes, use the `quotesHistorical()` or `ohlcvHistorical()` methods to fetch historical price data.

**Q: How many cryptocurrencies are supported?**
A: CoinMarketCap tracks 10,000+ cryptocurrencies including Bitcoin, Ethereum, and all major altcoins.

**Q: Can I convert between different fiat currencies?**
A: Yes, use the `convert` parameter with currencies like USD, EUR, GBP, JPY, etc.

### Technical Questions

**Q: What PHP version is required?**
A: PHP 8.1 or higher is required to use modern PHP features like readonly properties and union types.

**Q: Does this work with Laravel 11?**
A: Yes, the package supports both Laravel 10.x and 11.x.

**Q: Can I customize the HTTP client?**
A: Yes, the package uses PSR-18 standard, allowing you to inject any compatible HTTP client.

**Q: How do I handle API errors?**
A: The package provides specific exceptions for different error types. See the Error Handling section above.

### Performance & Optimization

**Q: How can I reduce API calls?**
A: Implement caching, use batch requests, and store frequently accessed data in your database.

**Q: What's the best caching strategy?**
A: Cache cryptocurrency listings for 5 minutes, individual prices for 1 minute, and metadata for 1 hour.

**Q: Can I use this in a high-traffic application?**
A: Yes, with proper caching and a suitable CoinMarketCap API plan. Consider upgrading to Professional plan for
high-volume applications.

## Integration examples

### WordPress Plugin Integration

```php
// WordPress plugin example
add_shortcode('crypto_price', function($atts) {
    $symbol = $atts['symbol'] ?? 'BTC';
    
    $client = (new ClientBuilder())
        ->setApiKey(get_option('coinmarketcap_api_key'))
        ->build();
    
    $quote = $client->cryptocurrency()->quotesLatest([
        'symbol' => $symbol,
        'convert' => 'USD'
    ]);
    
    $price = $quote['data'][$symbol]['quote']['USD']['price'];
    
    return sprintf('<span class="crypto-price">$%s</span>', number_format($price, 2));
});

// Usage: [crypto_price symbol="BTC"]
```

### API Endpoint for Mobile Apps

```php
// routes/api.php
Route::get('/crypto/prices', function () {
    $prices = Cache::remember('mobile_crypto_prices', 60, function () {
        return CoinMarketCap::cryptocurrency()->listingsLatest([
            'limit' => 50,
            'convert' => 'USD'
        ]);
    });
    
    return response()->json($prices);
});
```

### Webhook Integration

```php
// Send crypto price updates via webhook
class CryptoPriceWebhook
{
    public function sendPriceUpdate($symbol)
    {
        $quote = CoinMarketCap::cryptocurrency()->quotesLatest([
            'symbol' => $symbol,
            'convert' => 'USD'
        ]);
        
        $data = [
            'symbol' => $symbol,
            'price' => $quote['data'][$symbol]['quote']['USD']['price'],
            'change_24h' => $quote['data'][$symbol]['quote']['USD']['percent_change_24h'],
            'timestamp' => now()->toIso8601String()
        ];
        
        Http::post('https://your-webhook-url.com/crypto-update', $data);
    }
}
```

## Testing

### Run tests

```bash
composer test
```

### Coverage report

```bash
composer test-coverage
```

### Static analysis (PHPStan level 8)

```bash
composer phpstan
```

### Code style (PSR-12)

```bash
composer cs-fix
```

### Writing tests

```php
namespace Tigusigalpa\CoinMarketCap\Tests\Unit;

use PHPUnit\Framework\TestCase;
use Tigusigalpa\CoinMarketCap\ClientBuilder;

class YourFeatureTest extends TestCase
{
    public function test_your_feature_works()
    {
        $client = (new ClientBuilder())
            ->setApiKey('test-key')
            ->build();
        
        // Your test assertions
        $this->assertInstanceOf(Client::class, $client);
    }
}
```

## Contributing

1. **Fork the Repository**
   ```bash
   git clone https://github.com/tigusigalpa/coinmarketcap.git
   cd coinmarketcap
   ```

2. **Install Dependencies**
   ```bash
   composer install
   ```

3. **Create a Feature Branch**
   ```bash
   git checkout -b feature/add-new-endpoint
   ```

4. **Make Your Changes**
    - Write clean, well-documented code
    - Follow PSR-12 coding standards
    - Add type hints to all methods
    - Use strict types declaration

5. **Write Tests**
   ```bash
   # Add tests in tests/Unit or tests/Feature
   composer test
   ```

6. **Run Quality Checks**
   ```bash
   composer phpstan      # Static analysis
   composer cs-fix       # Code style
   composer test         # All tests
   ```

7. **Commit Your Changes**
   ```bash
   git commit -m "Add support for new CoinMarketCap endpoint"
   ```

8. **Push and Create Pull Request**
   ```bash
   git push origin feature/add-new-endpoint
   ```

### Guidelines

- Follow PSR-12, use PHP 8.1+ features, PHPDoc blocks on public methods
- PHPStan level 8 must pass
- Write unit tests for new features
- Update README and CHANGELOG as needed

## Security

Do **not** create public GitHub issues for security vulnerabilities. Email: **sovletig@gmail.com**

## License

MIT License — see [LICENSE](LICENSE).

## Author

**Igor Sazonov**
- GitHub: [@tigusigalpa](https://github.com/tigusigalpa)
- Email: sovletig@gmail.com

## Links

- [CoinMarketCap API Documentation](https://coinmarketcap.com/api/documentation/v1/)
- [CoinMarketCap API Portal](https://coinmarketcap.com/api/)
- [GitHub Repository](https://github.com/tigusigalpa/coinmarketcap)
- [Packagist](https://packagist.org/packages/tigusigalpa/coinmarketcap)
- [Changelog](CHANGELOG.md)
