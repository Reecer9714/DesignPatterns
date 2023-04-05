# Lesson: Observer Design Pattern
## Introduction

The Observer Design Pattern is a design pattern used to establish a one-to-many relationship between objects. In this relationship, when one object changes state, all its dependents are notified and updated automatically. The Observer pattern is used extensively in event-driven systems and is particularly useful in decoupling the sender and receiver of events.
## Example Scenario

Suppose we have a weather station that measures the temperature, humidity, and pressure. We want to display this information on a variety of displays, such as a current conditions display, a forecast display, and a statistics display. We need a way for the displays to be notified when the weather data changes, without tightly coupling the displays to the weather station.
## Design

UML Diagram
Here's the UML diagram for the Observer Design Pattern:

Observer Design Pattern UML Diagram

+ Subject: the object being observed, which maintains a list of its observers and notifies them of any state changes.
+ Observer: the interface or abstract class that defines the method(s) that are called when the subject's state changes.
+ ConcreteSubject: a subclass of Subject that implements the notifyObservers() method, which calls the update() method of each observer when the subject's state changes.
+ ConcreteObserver: a subclass of Observer that implements the update() method, which is called by the subject when its state changes.
## Implementation

To implement the Observer Design Pattern, we start by defining the observer interface. The observer interface defines a method for receiving updates:

```c++
template <typename T>
class Observer {
public:
    virtual void update(const T& data) = 0;
};
```

Next, we define the subject abstract class, which maintains a list of observers and provides methods for attaching and detaching observers:

```c++
template <typename T>
class ObservableSubject {
public:
    virtual void addObserver(std::shared_ptr<Observer<T>> observer) {
        observers.push_back(observer);
    }

    virtual void removeObserver(std::shared_ptr<Observer<T>> observer) {
        observers.erase(std::remove(observers.begin(), observers.end(), observer), observers.end());
    }

    virtual void notifyObservers(const T& data) {
        for (const auto& observer : observers) {
            observer->update(data);
        }
    }

private:
    std::vector<std::shared_ptr<Observer<T>>> observers;
};
```

We can then define a concrete subject class that implements the subject interface and maintains the state of the subject:

```c++
class WeatherData : public ObservableSubject<WeatherInfo> {
public:
    struct WeatherInfo {
        float temperature;
        float humidity;
        float pressure;
    };

    void setMeasurements(float temperature, float humidity, float pressure) {
        WeatherInfo data{temperature, humidity, pressure};
        weatherInfo = data;
        measurementsChanged();
    }

    void measurementsChanged() {
        notifyObservers(weatherInfo);
    }

private:
    WeatherInfo weatherInfo;
};
```

Finally, we define one or more concrete observer classes that implement the observer interface and maintain state related to the subject:

```c++
class CurrentConditionsDisplay : public Observer<WeatherData::WeatherInfo>, public std::enable_shared_from_this<CurrentConditionsDisplay> {
public:
    CurrentConditionsDisplay(std::shared_ptr<ObservableSubject<WeatherData::WeatherInfo>> weatherData)
        : weatherData(weatherData) {
        weatherData->addObserver(shared_from_this());
    }

    void update(const WeatherData::WeatherInfo& data) override {
        temperature = data.temperature;
        humidity = data.humidity;
        display();
    }

    void display() const {
        std::cout << "Current conditions: " << temperature << "F degrees and " << humidity << "% humidity\n";
    }

private:
    std::shared_ptr<ObservableSubject<WeatherData::WeatherInfo>> weatherData;
    float temperature;
    float humidity;
};
```
```c++
class StatisticsDisplay : public Observer<WeatherData::WeatherInfo>, public std::enable_shared_from_this<StatisticsDisplay> {
public:
    StatisticsDisplay(std::shared_ptr<ObservableSubject<WeatherData::WeatherInfo>> weatherData)
        : weather_station_(weather_station) {
        weather_station_.addObserver(shared_from_this());
    }

    void update() override {
        float temperature = weather_station_.getTemperature();
        if (temperature > max_temperature_) {
            max_temperature_ = temperature;
        }
        if (temperature < min_temperature_) {
            min_temperature_ = temperature;
        }
        temperature_sum_ += temperature;
        num_readings_++;
        display();
    }

    void display() const {
        std::cout << "Avg/Max/Min temperature: " << (temperature_sum_ / num_readings_) << "/" << max_temperature_ << "/" << min_temperature << "\n";
    }
private:
    std::shared_ptr<ObservableSubject<WeatherData::WeatherInfo>> weatherData;
    float min_temperature_;
    float max_temperature_;
    float temperature_sum_;
    int num_readings_;
};
```

## Usage
In this example scenario, we use the Observer Design Pattern to implement the weather data monitoring system. We create a `WeatherData` object and several `Observer` objects, passing the `WeatherData` object as a parameter to the `Observer` constructor. We could then update the stock price as necessary, and all the `Observer` objects would be notified and updated automatically.
```c++
int main() {
    auto weatherData = std::make_shared<WeatherStation>();
    auto currentDisplay = std::make_shared<CurrentConditionsDisplay>(weatherData);
    auto statisticsDisplay = std::make_shared<StatisticsDisplay>(weatherData);

    // change the weather data
    weatherData->measurementsChanged();

    // remove the observer
    weatherData->removeObserver(currentDisplay);

    // change the weather data again
    weatherData->measurementsChanged();
}
```

## Conclusion
The Observer Design Pattern is a powerful tool for decoupling objects and providing a way for objects to be notified of state changes. By using the Observer pattern, we can establish a one-to-many relationship between objects, making it easier to maintain the code and adding new functionality in the future.

## Additional Resources
+ http://codingadventures.org/2021/10/30/observer-pattern-in-modern-c/
+ http://www.vishalchovatiya.com/observer-design-pattern-in-modern-cpp/
+ https://sourcemaking.com/design_patterns/observer
