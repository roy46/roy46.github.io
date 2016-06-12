---
layout: post
title:  "How to MakeValid a DbGeometry"
date:   2016-06-05 13:45:00 +0100
categories: c# spatial DbGeometry SqlGeometry
---

Requires: Microsoft.SqlServer.Types
 
{% highlight C# linenos=table %}
public static class SpatialUtils
{
  public static DbGeometry MakeValid(this DbGeometry geom, int srid = 0)
  {
      if (geom.IsValid)
      {
          return geom;
      }

      return DbGeometry.FromText(SqlGeometry.STGeomFromText(new SqlChars(geom.AsText()), srid).MakeValid().STAsText().ToSqlString().ToString(), srid);
  }
}
{% endhighlight %} 
