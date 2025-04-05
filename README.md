# GoTravel
1. Concurrent Flight Search (Goroutines + Channels):
   
func SearchFlights(destinations []string) ([]Flight, error) {
    results := make(chan Flight, len(destinations))
    var wg sync.WaitGroup

    for _, dest := range destinations {
        wg.Add(1)
        go func(d string) {
            defer wg.Done()
            flight, err := AmadeusAPI.Search(d) // Third-party call
            if err == nil {
                results <- flight
            }
        }(dest)
    }

    wg.Wait()
    close(results)

    var flights []Flight
    for f := range results {
        flights = append(flights, f)
    }
    return flights, nil
}

2. Redis Caching Layer:
   
func GetCachedHotels(location string) ([]Hotel, error) {
    cacheKey := fmt.Sprintf("hotels:%s", location)
    cached, err := redisClient.Get(cacheKey).Bytes()
    
    if err == nil {
        var hotels []Hotel
        json.Unmarshal(cached, &hotels) // Cache hit
        return hotels, nil
    }

    hotels, err := Database.GetHotels(location) // Cache miss
    if err != nil {
        return nil, err
    }

    serialized, _ := json.Marshal(hotels)
    redisClient.Set(cacheKey, serialized, 10*time.Minute) // TTL
    return hotels, nil
}

3.Error Handling Middleware (Gin):

func ErrorHandler() gin.HandlerFunc {
    return func(c *gin.Context) {
        c.Next() // Process request
        
        if len(c.Errors) > 0 {
            err := c.Errors.Last()
            c.JSON(http.StatusInternalServerError, gin.H{
                "error":  "API_ERROR",
                "detail": err.Error(),
            })
        }
    }
}

4. Project Impact:
   
40% faster than legacy Python service.

Zero booking failures during Black Friday traffic spike.

5. How to Run:
   
git clone https://github.com/54N70SH/gotravel
docker-compose up  # Starts Postgres/Redis
go run main.go
