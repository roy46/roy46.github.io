---
layout: post
title:  "How to convert lat long to XYZ (ECEF)"
date:   2016-06-05 13:45:00 +0100
categories: c-sharp spatial
---

 
{% highlight C# linenos=table %}
public static class SpatialUtils
{
  public const int EarthRadius = 6378137; 

  public static double[] LatLongToXYZ(double lat, double lon, double elevation) 
  {        
      var cosLat = Math.Cos(lat * Math.PI / 180.0);
      var sinLat = Math.Sin(lat * Math.PI / 180.0);
      var cosLon = Math.Cos(lon * Math.PI / 180.0);
      var sinLon = Math.Sin(lon * Math.PI / 180.0);
      var rad = EarthRadius;
      var f = 1.0 / 298.257224;
      var C = 1.0 / Math.Sqrt(cosLat * cosLat + (1 - f) * (1 - f) * sinLat * sinLat); 
      var S = (1.0 - f) * (1.0 - f) * C;
      var h = elevation;
      var x = (rad * C + h) * cosLat * cosLon;
      var y = (rad * C + h) * cosLat * sinLon;
      var z = (rad * S + h) * sinLat;

      return new[] {x, y, z};
  }
}
{% endhighlight %} 