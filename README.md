# xlpp
Waziup Extended Low Power Payload (XLPP)

[![Documentation](https://godoc.org/github.com/waziup/xlpp?status.svg)](http://godoc.org/github.com/waziup/xlpp)

## Example

```go

package main

import (
	"log"

	"github.com/waziup/xlpp"
)

// LPP Types
var digitalInput = xlpp.DigitalInput(12)
var digitalOutput = xlpp.DigitalOutput(12)
var analogInput = xlpp.AnalogInput(3.75)
var analogOutput = xlpp.AnalogOutput(4.25)
var luminosity = xlpp.Luminosity(45)
var presence = xlpp.Presence(5)
var temperature = xlpp.Temperature(31.6)
var relativeHumidity = xlpp.RelativeHumidity(22.5)
var accelerometer = xlpp.Accelerometer{X: 3.245, Y: -0.171, Z: 0.909}
var barometricPressure = xlpp.BarometricPressure(4.1)
var gyromter = xlpp.Gyrometer{X: 4.25, Y: 5.1, Z: 0.2}
var gps = xlpp.GPS{Latitude: 51.0493, Longitude: 13.7381, Meters: 122}

// XLPP Types
var integer = xlpp.Integer(5182)
var str = xlpp.String("test :)")
var boolean = xlpp.Bool(true)
var bin = xlpp.Binary([]byte{1, 2, 3, 7, 8, 9})
var object = xlpp.Object{
	"count": &integer,
	"pos":   &gps,
	"val":   &digitalInput,
}
var array = xlpp.Array{
	&presence,
	&luminosity,
	&temperature,
}
var null = xlpp.Null{}


var values = []xlpp.Value{
	// LPP types
	&digitalInput, &digitalOutput, &analogInput, &analogOutput,
	&luminosity, &presence, &temperature, &relativeHumidity,
	&accelerometer, &barometricPressure, &gyromter, &gps,
	// XLPP types
	&integer, &str, &boolean, &bin, &object, &array, &null,
}


func main() {
	var buf bytes.Buffer

	// write types using xlpp.Writer
	w := xlpp.NewWriter(&buf)
	for i, value := range values {
		var channel = uint8(i)
		w.Add(channel, value)
    }
    
    log.Printf("buffer size: %d B", buf.Len())
    // > buffer size 130 B

	// read types using xlpp.Reader
	r := xlpp.NewReader(&buf)
	for {
		channel, value, err := r.Next()
		if err != nil {
			log.Fatal("reading error:", err)
		}
		if value == nil {
			log.Fatal("end")
			break
		}
		log.Printf("%2d: %v", channel, value)
	}
}

```


# LPP Types

Type | IPSO | LPP | Hex | Data Size | Data Resolution per bit
-- | -- | -- | -- | -- | --
Digital Input | 3200 | 0 | 0 | 1 | 1
Digital Output | 3201 | 1 | 1 | 1 | 1
Analog Input | 3202 | 2 | 2 | 2 | 0.01 Signed
Analog Output | 3203 | 3 | 3 | 2 | 0.01 Signed
Illuminance Sensor | 3301 | 101 | 65 | 2 | 1 Lux Unsigned MSB
Presence Sensor | 3302 | 102 | 66 | 1 | 1
Temperature Sensor | 3303 | 103 | 67 | 2 | 0.1 °C Signed MSB
Humidity Sensor | 3304 | 104 | 68 | 1 | 0.5 % Unsigned
Accelerometer | 3313 | 113 | 71 | 6 | 0.001 G Signed MSB per axis
Barometer | 3315 | 115 | 73 | 2 | 0.1 hPa Unsigned MSB
Gyrometer | 3334 | 134 | 86 | 6 | 0.01 °/s Signed MSB per axis
GPS Location | 3336 | 136 | 88 | 9 | Latitude : 0.0001 ° Signed MSB Longitude : 0.0001 ° Signed MSB Altitude : 0.01 meter Signed MSB


# XLPP

Type | XLPP | Data Size | Data Resolution per bit
-- | -- | -- | --
Null | 52 | 0 | (no value)
Bool | 53 | 1 | true of false
Integer | 51 | variant | 1
String | len(string)+1 | null terminated C string
Object | len(keys)+values+1 | keys are null terminated C strings, followed by the values
Array | len(values)+1 | list of values

# XLPP Binary

Install the xlpp binary from source using the [go programming language](https://golang.org/dl/):

```cmd
> go install github.com/waziup/xlpp/cmd/xlpp
```

Usage:

```
Encoding:
> xlpp ./xlpp -e '{"temperature0":23.5}'
< AGcA6w==

Decoding:
> xlpp ./xlpp -d AGcA6w==
< {"temperature0":23.5}
```

# References

Based on [Cayenne Low Power Payload](https://www.thethingsnetwork.org/docs/devices/arduino/api/cayennelpp.html). See [developers.mydevices.com/cayenne](https://developers.mydevices.com/cayenne/docs/lora/#lora-cayenne-low-power-payload)