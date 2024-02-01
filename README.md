// Define input parameters
input int StochasticPeriod = 14;
input int StochasticDPeriod = 3;
input int StochasticKPeriod = 3;
input int ConfirmationCandle = 2;
input int DailyStochConfirmationPeriod = 14;
input int H4StochConfirmationPeriod = 14;
input int TakeProfitPips = 50;
input int StopLossPips = 20;
input double LotSize = 0.1;

// Define global variables
int ticket;

// Define trading function
int start() {
    // Check if the current timeframe is Daily
    if (Period() != PERIOD_D1) {
        return(0);
    }

    // Calculate Daily Stochastic values
    double dailyStoch = iStochastic(Symbol(), PERIOD_D1, StochasticKPeriod, StochasticDPeriod, StochasticPeriod, MODE_MAIN, 0, MODE_MAIN, 0, MODE_MAIN, 0);

    // Check for a bullish signal on the Daily chart
    if (dailyStoch < 30) {
        // Check for confirmation candle
        if (iClose(Symbol(), PERIOD_D1, ConfirmationCandle) > iOpen(Symbol(), PERIOD_D1, ConfirmationCandle)) {
            // Check if Stochastic lines spread and head upward
            if (iStochastic(Symbol(), PERIOD_D1, StochasticKPeriod, StochasticDPeriod, StochasticPeriod, MODE_MAIN, 0, 0) > iStochastic(Symbol(), PERIOD_D1, StochasticKPeriod, StochasticDPeriod, StochasticPeriod, MODE_MAIN, 1, 0)) {
                // Open a buy trade on the 4-hour chart
                OpenBuyTrade();
            }
        }
    }

    // Check for a bearish signal on the Daily chart
    if (dailyStoch > 70) {
        // Check for confirmation candle
        if (iClose(Symbol(), PERIOD_D1, ConfirmationCandle) < iOpen(Symbol(), PERIOD_D1, ConfirmationCandle)) {
            // Check if Stochastic lines spread and head downward
            if (iStochastic(Symbol(), PERIOD_D1, StochasticKPeriod, StochasticDPeriod, StochasticPeriod, MODE_MAIN, 0, 0) < iStochastic(Symbol(), PERIOD_D1, StochasticKPeriod, StochasticDPeriod, StochasticPeriod, MODE_MAIN, 1, 0)) {
                // Open a sell trade on the 4-hour chart
                OpenSellTrade();
            }
        }
    }

    return(0);
}

// Function to open a buy trade on the 4-hour chart
void OpenBuyTrade() {
    // Calculate 4-hour Stochastic values
    double h4Stoch = iStochastic(Symbol(), PERIOD_H4, StochasticKPeriod, StochasticDPeriod, StochasticPeriod, MODE_MAIN, 0, MODE_MAIN, 0, MODE_MAIN, 0);

    // Open a buy trade on the 4-hour chart when Stochastic is below 30
    if (h4Stoch < 30) {
        ticket = OrderSend(Symbol(), OP_BUY, LotSize, Ask, 3, 0, 0, "Buy Trade", 0, 0, Green);
        // Set take profit and stop loss
        if (ticket > 0) {
            OrderSend(Symbol(), OP_TAKEPROFIT, Ask + TakeProfitPips * Point, 3, 0, 0, 0, "Take Profit", 0, 0, Green);
            
            // Set stop loss based on the 4-hour Stochastic intersection
            double stopLossLevel = iStochastic(Symbol(), PERIOD_H4, StochasticKPeriod, StochasticDPeriod, StochasticPeriod, MODE_MAIN, 0, 0);
            OrderSend(Symbol(), OP_SL, stopLossLevel, 3, 0, 0, 0, "Stop Loss", 0, 0, Red);
        }
    }
}

// Function to open a sell trade on the 4-hour chart
void OpenSellTrade() {
    // Calculate 4-hour Stochastic values
    double h4Stoch = iStochastic(Symbol(), PERIOD_H4, StochasticKPeriod, StochasticDPeriod, StochasticPeriod, MODE_MAIN, 0, MODE_MAIN, 0, MODE_MAIN, 0);

    // Open a sell trade on the 4-hour chart when Stochastic is above 70
    if (h4Stoch > 70) {
        ticket = OrderSend(Symbol(), OP_SELL, LotSize, Bid, 3, 0, 0, "Sell Trade", 0, 0, Red);
        // Set take profit and stop loss
        if (ticket > 0) {
            OrderSend(Symbol(), OP_TAKEPROFIT, Bid - TakeProfitPips * Point, 3, 0, 0, 0, "Take Profit", 0, 0, Red);
            
            // Set stop loss based on the 4-hour Stochastic intersection
            double stopLossLevel = iStochastic(Symbol(), PERIOD_H4, StochasticKPeriod, StochasticDPeriod, StochasticPeriod, MODE_MAIN, 0, 0);
            OrderSend(Symbol(), OP_SL, stopLossLevel, 3, 0, 0, 0, "Stop Loss", 0, 0, Green);
        }
    }
}
