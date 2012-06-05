---
date: '2009-04-22 15:39:14'
layout: post
slug: php-named-parameters
status: publish
title: PHP Named Parameters
wordpress_id: '227'
categories:
- Development
- PHP
tags:
- named parameters
- php
---

I'm not sure why I haven't posted this yet.  The following code block is (one way) of simulating named parameters in PHP for class method calls.  It utilizes PHP's magic method **__call** and takes advantage of PHP 5's Reflection API for determining default values of parameters not passed.

Parameters can be passed two ways:

```php
$obj->method(array('key' => 'value', 'key2' => 'value2'));
```

```php
$obj->method(':key = value', ':key2 = value2');
```

Also, if using the latter (preferred) method of parameter passing you can define one parameter as an array in your method declaration and this will intelligently handle that as well.  The TestClass below is an example of this.

```php
$tc = new TestClass;
$tc->test(':a = testing', ':b = true', array('p', 'h', 'p'));

class TestClass extends NamedParameters
{
	public function _test($a = 'test', $b = false, $c = array())
	{
		echo '$a = '.$a."\n";
		echo '$b = '.($b ? 'true' : 'false').' ('.gettype($b).")\n";
		echo "\$c:\n";
		foreach($c as $k => $v) {
			echo '   '.$k.' = '.$v."\n";
		}
	}
}
```

And finally, here's the implementation:

```php
class NamedParameters
{
	/**
	 * Implementation of PHP's magic method __call to support named parameters
	 *
	 * Actual helper method names that want to use named parameters are 
	 * prefixed with _ so calls are redirected through this method. It
	 * uses PHP's reflection api to simulate named parameters.
	 *
	 * Named parameters can be passed in one of two ways.
	 * 		$obj->method(array('key' => 'value', 'key2' => 'value2'));
	 * or
	 *		$obj->method(':key = value', ':key2 = value2');
	 *
	 * @param string $n name of method application actually called
	 * @param array $a parameters passed to method
	 * @return mixed output of method call
	 * @access public
	 */
	public function __call($n, $a)
	{
		if(method_exists($this, '_'.$n)) {
			$methodParams = array();
			$passedParams = array();
			
			$reflectMethod = new ReflectionMethod(get_class($this), '_'.$n);
			if(isset($a[0]) && is_array($a[0])) {
				// first parameter passed is an array, so we assume all parameters are in this array
				$passedParams = $a[0];
			} else {
				// passing parameters as strings using named parameter syntax
				foreach($a as $v) {
					if(is_string($v)) {
						// format is ':parameterName = parameterValue'
						if(preg_match("/^:([a-z0-9]+)\s*=\s*(.+)$/isD", $v, $out)) {
							$passedParams[$out[1]] = $out[2];
						}
					} elseif(is_array($v)) {
						// technique allows one parameter to be an "array" parameter
						$passedParams['__array__'] = $v;
					}
				}
			}
			
			// loop through the parameters of the function being called
			foreach($reflectMethod->getParameters() as $i => $param) {
				$defaultAvailable = $param->isDefaultValueAvailable();
				$parameterName = $param->getName();
				if($defaultAvailable) {
					$default = $param->getDefaultValue();
					if(($paramType = gettype($default)) == 'array') {
						// match this parameter of the function being called to the array 
						// that was passed, if any (see above)
						$parameterName = '__array__';
					}
				}
				
			
				if(array_key_exists($parameterName, $passedParams)) {
					// this parameter of the function being called was passed in the named parameters
					$val = $passedParams[$parameterName];
					if($defaultAvailable && is_string($val)) {
						// if the function being called specified default values for this parameter
						// we can type cast
						switch($paramType) {
							case 'boolean':
								$val = (strtolower($val) == 'true') ? true : false;
								break;
							case 'integer':
								$val = intval($val);
								break;
							case 'double':
								$val = floatval($val);
								break;
							default:
								break;
						}
					}
					$methodParams[] = $val;
				} elseif($defaultAvailable) {
					// parameter was not passed, assign a default value
					$methodParams[] = $default;
				} else {
					// parameter was not passed and no default value exists, trigger an error
					trigger_error("Beast __call to '".get_class($this)."::".$n."' missing parameter #".($i+1)." (".$parameterName.")", E_USER_ERROR);
					return null;
				}
			}
			
			// call the function and direct output through handler
			return $this->output(call_user_func_array(array(&$this, '_'.$n), $methodParams), isset($passedParams['return']) ? $passedParams['return'] : false);
		} else {
			// method doesn't exist, trigger an error
			trigger_error('Beast __call to a non-existant method '.get_class($this).'::'.$n, E_USER_ERROR);
			return null;
		}
	}
}
```

Hope you find this useful!
