[![Build Status](https://dev.azure.com/andreasmewald/Exceptions/_apis/build/status/moerwald.Exceptions?branchName=master)](https://dev.azure.com/andreasmewald/Exceptions/_build/latest?definitionId=1?branchName=master)
[![](https://img.shields.io/azure-devops/tests/andreasmewald/Exceptions/1.svg)](https://dev.azure.com/andreasmewald/Exceptions)
[![](https://img.shields.io/azure-devops/coverage/andreasmewald/Exceptions/1.svg)](https://dev.azure.com/andreasmewald/Exceptions/_build?definitionId=1)
[![](https://img.shields.io/nuget/dt/Mwd.Exceptions.svg)](Mwd.Exceptions)
[![](https://img.shields.io/nuget/v/Mwd.Exceptions.svg)](https://www.nuget.org/packages/Mwd.Exceptions)
[![Quality Gate Status](https://sonarcloud.io/api/project_badges/measure?project=moerwald_Exceptions&metric=alert_status)](https://sonarcloud.io/dashboard?id=moerwald_Exceptions)
[![Code Smells](https://sonarcloud.io/api/project_badges/measure?project=moerwald_Exceptions&metric=code_smells)](https://sonarcloud.io/dashboard?id=moerwald_Exceptions)
[![Lines of Code](https://sonarcloud.io/api/project_badges/measure?project=moerwald_Exceptions&metric=ncloc)](https://sonarcloud.io/dashboard?id=moerwald_Exceptions)
[![Maintainability Rating](https://sonarcloud.io/api/project_badges/measure?project=moerwald_Exceptions&metric=sqale_rating)](https://sonarcloud.io/dashboard?id=moerwald_Exceptions)
[![Security Rating](https://sonarcloud.io/api/project_badges/measure?project=moerwald_Exceptions&metric=security_rating)](https://sonarcloud.io/dashboard?id=moerwald_Exceptions)
[![BCH compliance](https://bettercodehub.com/edge/badge/moerwald/Exceptions?branch=master)](https://bettercodehub.com/)

# Exceptions

This module offers the following exception helpers:

- Boundary
- Convert
- Ignore

## Boundary

Since a lot of exceptions are undocumented in the NET framework or third party libraries code ends up with a lot of

``` C#

try{

}
catch (Exception ex){

}

```

blocks. In my oppionion these blocks should only work as exception boundary. Since it is unclear if a `try-catch`-anything-block works as exception boundary, an explicit "statement" would be an improvement. The static `Boundary` class offers such an statement:

```C#

    [Test]
    public void CatchDoesntPopulateException()
    {
      Assert.DoesNotThrow(
        () => Boundary.CatchAll(
          () => throw new Exception("Test")
        )
      );
    }

```

As the name says `CatchAll` catches all kind of exceptions. `CatchAll` accepts an additional (optional) delegate populating the cought exception to e.g. log it to a file.

## Convert

The `Convert` class translates `ExceptionA` to `ExceptionB`. This behaviour is usefull to translate a standard exception to a custom one. Example: You've a config class, every error during config loading, parsing, validating shall be communicated via an `ConfigException`. Based on that you're able to e.g. wrap a `FileNotFound`-exception in a `ConfigException` (where the inner exeption is `FileNotFound`).

Example:

```C#
        [Test]
        public void ConvertExceptionToMyCustomException()
        {
            var caughtException = Assert.Throws<MyCustomException>(() =>
            {
                Exceptions.Convert.To<ArgumentNullException, MyCustomException>(() => throw new ArgumentNullException("Test"));
            });

            Assert.Multiple(() =>
            {
                Assert.That(caughtException.InnerException, Is.Not.Null);
                Assert.That(caughtException.InnerException.GetType(), Is.EqualTo(typeof(ArgumentNullException)));
            });
        }

```

# Ignore

Sometimes you only want to work with return types, and don't bother any exception. The `Ignore` class offers an `AllExceptions` and `SomeExceptions` method (which are self-explanatory).

```C#

        [Test]
        public void IgnoreExceptionAndReturnNativeType()
        {
            Assert.DoesNotThrow(() =>
            {
                var result = Ignore.AllExceptions<bool>(() => throw new Exception("Test"));
                Assert.That(result, Is.False);
            });
        }

```

In case of an exception `AllExceptions<bool>` returns the default value of the given generic type (in case of bool `false`). As other methods `AllExceptions<bool>` offers an delegate to e.g. log the occurred exceptions. `Ignore`class also offers a `SomeExceptions`method:

```C#
  [Test]
        public void IgnoreSomeExceptionsCaughtExceptionIsIgnored()
        {
            Assert.DoesNotThrow(
                () =>
                {
                    var result = Ignore.SomeExceptions<bool, ArgumentException, NullReferenceException, FormatException>(() =>
                    {
                        throw new ArgumentException("Test");
                    });

                    Assert.That(result, Is.False);
                }
                );
        }
```


