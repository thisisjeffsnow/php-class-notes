# PHP II - April 2022

## TODO
* Q: Can you use the keyword "new" in property or const definition in 8.1?
* Q: Locate original article on why `__unserialize()` was introduced

## Homework
For Wed 20 Apr 2022
* Lab: Namespace
* Lab: Create a Class
* Lab: Create an Extensible Super Class [optional]
For Fri 22 Apr 2022
* https://collabedit.com/5duxj

## Resources
Code examples: https://github.com/dbierer/classic_php_examples

## Class Notes

### Namespaces
Class example:
```
<?php
namespace My\Different\Space;

// you can identify PHP classes like this:
use ArrayObject;
class Test
{
    public function test()
    {
        return __NAMESPACE__;
    }
    public function getArrayObject(array $arr)
    {
    	// or: use leading backslash:
    	return new \ArrayObject($arr);
    }
}
```
Calling program:
```
<?php
include __DIR__ . '/Test.php';
use My\Different\Space\Test;

// alternatively, you can do this:
// $test = new \My\Different\Space\Test();

$test = new Test();
echo $test->test();

$arr = [1,2,3,4,5];
$obj = $test->getArrayObject($arr);
var_dump($obj);
```
Call methods from inside a class: use `$this`
```
<?php
class Test
{
	protected string $first = 'fred';
	protected string $last  = 'flintstone';
	public function getFirst()
	{
		return ucfirst($this->first);
	}
	public function getLast()
	{
		return ucfirst($this->last);
	}
	public function getName()
	{
		return $this->getFirst() . ' ' . $this->getLast();
	}
}
$test = new Test();
echo $test->getName();
```
Constructor argument promotion example:
```
<?php
class Test
{
	// contructor argument promotion
	// only in PHP 8.0 and above
	public function __construct(public string $first = '', public string $last = '')
	{
		// do nothing
	}
	public function getFirst()
	{
		return ucfirst($this->first);
	}
	public function getLast()
	{
		return ucfirst($this->last);
	}
	public function getName()
	{
		return $this->getFirst() . ' ' . $this->getLast() . "\n";
	}
}
$test1 = new Test('Fred', 'Flintstone');
echo $test1->getName();

$test2 = new Test('Wilma', 'Flintstone');
echo $test2->getName();

var_dump($test1, $test2);
```
## Magic Methods
Example of `__destruct()`
* https://github.com/dbierer/filecms-core/blob/main/src/Common/Image/Captcha.php
Example of `__sleep()` and `__wakeup()`
```
<?php
class UserEntity {
	public $hash = '';
    public function __construct(
        protected string $firstName,
        protected string $lastName)
    {
		$this->hash = bin2hex(random_bytes(8));
    }
    public function __sleep()
    {
		return ['firstName','lastName'];
	}
	public function __wakeup()
	{
		$this->hash = bin2hex(random_bytes(8));
	}
	public function getFullName()
	{
		return $this->firstName . ' ' . $this->lastName;
	}
    public function getNativeString(): string {
        return serialize($this);
    }
}
$userEntity = new UserEntity('Mark', 'Watney');
var_dump($userEntity);
echo PHP_EOL;

$native = $userEntity->getNativeString();
$obj  = unserialize($native);
echo $native;
echo PHP_EOL;
var_dump($obj);
```
Example of `__serialize()` and `__unserialize()`
```
<?php
class UserEntity {
	public $hash = '';
    public function __construct(
        protected string $firstName,
        protected string $lastName)
    {
		$this->hash = bin2hex(random_bytes(8));
    }
    public function __serialize()
    {
		return [
			'firstName' => $this->firstName,
			'lastName' => $this->lastName,
			'date' => date('Y-m-d H:i:s')];
	}
	public function __unserialize($array)
	{
		// $array contains values restored from the serialization
		$this->hash = bin2hex(random_bytes(8));
	}
	public function getFullName()
	{
		return $this->firstName . ' ' . $this->lastName;
	}
    public function getNativeString(): string {
        return serialize($this);
    }
}
$userEntity = new UserEntity('Mark', 'Watney');
var_dump($userEntity);
echo PHP_EOL;

$native = $userEntity->getNativeString();
$obj  = unserialize($native);
echo $native;
echo PHP_EOL;
var_dump($obj);
```
Example of when getters and setters are useful:
```
class Test
{
    protected $date;
    const DEFAULT_FMT = 'Y-m-d H:i:s';
    // use getters and setters if there's a need for further processing
    public function setDate(string $str) : void
    {
		$this->date = new DateTime($str);
	}
	public function getDate(string $fmt = self::DEFAULT_FMT) : string
	{
		return $this->date->format($fmt);
	}
}
```
Example of Abstract class with abstract method:
* https://github.com/laminas/laminas-mvc/blob/master/src/Controller/AbstractController.php