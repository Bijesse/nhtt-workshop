# Lab 3: Building spans

In this lab, you are going to manually build spans so that you have deeper visibility into your code from the data in New Relic.

## Speaking with your developer
In the previous excercise, you were able to leverage New Relic to locate the problem. However, it took you much longer than you would have hoped to find a function that one of your colleagues probably forgot to remove while debugging an issue that was clearly not meant for production. :facepalm: 

You show a developer on your team the New Relic permalink for the distributed trace from Part 2 of this workshop. You task them with creating a solution that would allow you to get closer to identifying the exact line of code that is causing an issue or delay in this microservice. 

Your developer sent you a few code snippets at 9pm last night since they have a planned PTO for the next couple of weeks. Their message is below.


> Hey there,  
> 
> These updated code blocks will generate individual spans to fix your problem. Add them to each function in the main.go file along with the sleep commands you found earlier (to reproduce the issue you had previously) and you should see how I fixed the problem.
> 
> If you have any questions while I'm gone... good luck! You'll figure it out.
> 
> Cheers,  
> Your Favorite Dev


```
func (s *server) GetQuote(ctx context.Context, in *pb.GetQuoteRequest) (*pb.GetQuoteResponse, error) {

    log.Info("[GetQuote] received request")
    defer log.Info("[GetQuote] completed request")

    // NHTT Workshop - Building Spans
    quote := CreateQuoteFromCount(0, ctx)

    // Generate a response.
    return &pb.GetQuoteResponse{
        CostUsd: &pb.Money{
            CurrencyCode: "USD",
            Units:        int64(quote.Dollars),
            Nanos:        int32(quote.Cents * 10000000)},
    }, nil
}
```


```
func CreateQuoteFromCount(count int, ctx context.Context) Quote {

    // NHTT Workshop - Building Spans
    ctx, childSpan := tracer.Start(ctx, "CreateQuoteFromCount")
    defer childSpan.End()

    // NHTT Workshop - Adding a Delay
    time.Sleep(time.Second * 1)

    // NHTT Workshop - Building Spans
    return CreateQuoteFromFloat(float64(rand.Intn(100)), ctx)
}
```

```
func CreateQuoteFromFloat(value float64, ctx context.Context) Quote {

    // NHTT Workshop - Building Spans
    ctx, childSpan := tracer.Start(ctx, "CreateQuoteFromFloat")
    defer childSpan.End()

    // NHTT Workshop - Adding a Delay
    time.Sleep(time.Second * 3)

    units, fraction := math.Modf(value)
    return Quote{
        uint32(units),
        uint32(math.Trunc(fraction * 100)),
    }
}
```

## Your task
Read the message from the developer so that you can update the code in the `shippingservice/main.go` file. After you have updated the code, head back to New Relic to see how new distributed traces coming in have changed in terms of the additional information they provide. Document your findings.

**Note:**  You may need to wait 2-5 minutes before the changes are reflected in new traces you can view on New Relic
***

## Moving forward in the workshop
Once you have identified the benefit(s) of building spans, be sure to document your process. When that is done, you are ready to move onto the next lab in this workshop [Lab 4: Adding Span Attributes](lab_4-Span-Attributes.md) 
