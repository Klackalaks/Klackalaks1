# Klackalaks1
BOT
// Definición de parámetros
input int FastMA_Period = 10; // Período de la media móvil rápida
input int SlowMA_Period = 30; // Período de la media móvil lenta
input double DesiredProfitPercentage = 3.0; // Porcentaje de ganancia deseado por operación
input double MaxLossPercentage = 1.0; // Porcentaje máximo de pérdida por operación
input double MaxDailyLossPercentage = 8.0; // Porcentaje máximo de pérdida diaria
input double MaxDailyProfitPercentage = 30.0; // Porcentaje máximo de ganancia diaria
input double MaxRiskPercentage = 2.0; // Porcentaje máximo del capital en riesgo
input double LotSize = 0.1; // Tamaño de lote por operación
input int RSI_Period = 14; // Período del indicador RSI
input int RSI_Overbought = 70; // Nivel de sobrecompra del RSI
input int RSI_Oversold = 30; // Nivel de sobreventa del RSI

double AccountBalance = AccountBalance();
double TotalLossToday = 0.0;
double TotalProfitToday = 0.0;
datetime LastTradeTime = 0;

// Resto del código igual al ejemplo anterior...

void OnTick()
{
    // Verificar si el símbolo actual es EUR/USD
    if (Symbol() != "EURUSD")
    {
        Print("Este robot solo opera con el instrumento EUR/USD.");
        return;
    }
    
    // Verificar restricciones de pérdida y ganancia diarias
    if (TimeCurrent() - LastTradeTime < PeriodSeconds(DAILY))
    {
        if (TotalLossToday > (AccountBalance * (MaxDailyLossPercentage / 100.0)))
        {
            Print("Se ha alcanzado el límite máximo de pérdida diaria.");
            return;
        }
        
        if (TotalProfitToday > (AccountBalance * (MaxDailyProfitPercentage / 100.0)))
        {
            Print("Se ha alcanzado el límite máximo de ganancia diaria.");
            return;
        }
    }
    else
    {
        TotalLossToday = 0.0;
        TotalProfitToday = 0.0;
        LastTradeTime = TimeCurrent();
    }
    
    // Calcular el valor del RSI
    double rsiValue = iRSI(Symbol(), 0, RSI_Period, PRICE_CLOSE, 0);
    
    // Resto del código igual al ejemplo anterior...
    
    if (fastMA_value > slowMA_value && rsiValue < RSI_Oversold)
    {
        // Abrir una posición de compra
        double openPrice = SymbolInfoDouble(Symbol(), SYMBOL_BID);
        double maxLotSize = CalculateMaxLotSize(openPrice, OP_BUY);
        int ticket = OrderSend(Symbol(), OP_BUY, maxLotSize, openPrice, 2, 0, 0, "", 0, clrNONE);
        
        if (ticket > 0)
        {
            double takeProfit = CalculateTakeProfit(openPrice, OP_BUY);
            double stopLoss = CalculateMaxLoss(openPrice, OP_BUY);
            OrderSend(Symbol(), OP_BUY, maxLotSize, openPrice, 2, stopLoss, takeProfit, "", 0, clrNONE);
            TotalProfitToday += maxLotSize * (takeProfit - openPrice);
            Print("Operación de compra abierta en el precio: ", openPrice);
        }
        else
        {
            Print("Error al abrir la operación de compra: ", GetLastError());
        }
    }
    else if (fastMA_value < slowMA_value && rsiValue > RSI_Overbought)
    {
        // Abrir una posición de venta
        double openPrice = SymbolInfoDouble(Symbol(), SYMBOL_ASK);
        double maxLotSize = CalculateMaxLotSize(openPrice, OP_SELL);
        int ticket = OrderSend(Symbol(), OP_SELL, maxLotSize, openPrice, 2, 0, 0, "", 0, clrNONE);
        
        if (ticket > 0)
        {
            double takeProfit = CalculateTakeProfit(openPrice, OP_SELL);
            double stopLoss = CalculateMaxLoss(openPrice, OP_SELL);
            OrderSend(Symbol(), OP_SELL, maxLotSize, openPrice, 2, stopLoss, takeProfit, "", 0, clrNONE);
            TotalProfitToday += maxLotSize * (openPrice - takeProfit);
            Print("Operación de venta abierta en el precio: ", openPrice);
        }
        else
        {
            Print("Error al abrir la operación de venta: ", GetLastError());
        }
    }
}
