---
layout: post
title:  "How to MakeValid a DbGeometry"
date:   2016-06-05 13:45:00 +0100
categories: c# spatial DbGeometry SqlGeometry
disqus: true
live: true
---


In this post we'll discuss 

Entity framework's DbGeometry and DbGeorgraphy provide a lot of the functiona you find with sql server spatial. However some functions aren't available. 

One such function is the MakeValid. The below extension method is one way to implement the MakeValid. It does however require the SQL spatial library. 

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
