# X MessageFormat Objects

## X.1 Abstract Operations for MessageFormat Objects

### X.1.1 InitializeMessageFormat ( _messageFormat_, _locales_, _options_ )

The abstract operation InitializeMessageFormat accepts the arguments _messageFormat_ (which must be an object), _locales_, and _options_. It initializes _messageFormat_ as a MessageFormat object. The following steps are taken:

1. Let _requestedLocales_ be ? CanonicalizeLocaleList(_locales_).
2. Set _messageformat_.\[\[RequestedLocales]] to _requestedLocales_.
3. Set _options_ to ? CoerceOptionsToObject(_options_).
4. Let _matcher_ be ? GetOption(_options_, **"localeMatcher"**, **"string"**, « **"lookup"**, **"best fit"** », **"best fit"**).
5. Set _messageformat_.\[\[LocaleMatcher]] to matcher.
6. Let _select_ be ? GetMessageFormatRuntime(_options_, **"select"**, { \[\[plural]]: MessageSelectPlural }).
7. Set _messageFormat_.\[\[Select]] to _select_.
8. Let _format_ be ? GetMessageFormatRuntime(_options_, **"format"**, { \[\[datetime]]: MessageFormatDateTime, \[\[number]]: MessageFormatNumber }).
9. Set _messageFormat_.\[\[Format]] to _format_.
10. Return _messageFormat_.

### X.1.1 GetMessageFormatRuntime ( _options_, _property_, _runtime_ )

The abstract operation GetMessageFormatRuntime extracts the value of the property named _property_ from the provided _options_ object, checks that that it has the expected shape, and extends the base _runtime_ accordingly.

1. Assert: Type(_options_) is Object.
2. Let _O_ be ? Get(_options_, _property_).
3. If _O_ is **undefined**, return _runtime_.
4. Assert: Type(_O_) is Object.
5. Let _keys_ be ? _O_.\[\[OwnPropertyKeys]]().
6. For each element _key_ of _keys_, do:
    1. Let _value_ be Get(_O_, _key_).
    2. Assert: IsCallable(_value_) is **true**.
    3. Set _runtime_.\[\[\<key>]] to _value_.

### X.1.1 GetMessageContext ( _messageFormat_, _resId_, _msgPath_, _scope_ )

When the GetMessageContext abstract operation is called with the arguments _messageFormat_ (which must be an object initialized as a MessageFormat), _resId_, _msgPath_, and _scope_, the following steps are taken:

1. If Type(_msgPath_) is not String and IsArray(_msgPath_) is **false**, then
    1. Set _scope_ to _msgPath_.
    2. Set _msgPath_ to _resId_.
    3. Let _res0_ be _messageFormat_.\[\[Resources]]\[0].
    4. If _res0_ is **undefined**, throw a **RangeError** exception.
    5. Set _resId_ to Get(_res0_, **"id"**).
    6. For each element _res_ of _messageFormat_.\[\[Resources]], do
        1. Let _id_ be Get(_res_, **"id"**).
        2. If _id_ is not _resId_, throw a **RangeError** exception.
2. If Type(_resId_) is not String, throw a **TypeError** exception.
3. If Type(_msgPath_) is String, then
    1. Set _msgPath_ to ! CreateArrayFromList(« _msgPath_ »).
4. If _scope_ is **undefined**, then
    1. Set _scope_ to ! OrdinaryObjectCreate(**null**).
5. If Type(_scope_) is not Object, throw a **TypeError** exception.
6. Return a new Record { \[\[ResourceId]]: _resId_, \[\[MessagePath]]: _msgPath_, \[\[Scope]]: _scope_ }.

### X.1.1 GetMessageEntry ( _messageFormat_, _resId_, _msgPath_ )

When the abstract operation GetMessageEntry is called with arguments _messageFormat_ (which must be an object initialized as a MessageFormat), _resId_ (which must be a String value), and _msgPath_, it attempts to locate the corresponding Message or MessageGroup value among the known resources:

1. If IsArray(_msgPath_) is **false**, then
    1. Set _msgPath_ to CreateArrayFromList(« _msgPath_ »)
2. For each element _res_ of _messageFormat_.\[\[Resources]], do
    1. Let _id_ be Get(_res_, **"id"**).
    2. If _id_ is _resId_, then
        1. Let _msg_ be ? GetMessageEntryInResource(_res_, _msgPath_).
        2. If _msg_ is not **undefined**, return _msg_.
3. Return **undefined**.

### X.1.1 GetMessageEntryInResource ( _res_, _msgPath_ )

When the abstract operation GetMessageEntryInResource is called with arguments _res_ and _msgPath_ (which must be an Array of String values), it performs the following steps:

1. Let _msg_ be _res_
2. For each element _part_ of _msgPath_, do
    1. If _msg_ is **undefined**, return **undefined**.
    2. If Type(_msg_) is not Object, throw a **TypeError** exception.
    3. Let _entries_ be Get(_msg_, **"entries"**)
    4. If _entries_ is **undefined**, return **undefined**
    5. Assert: Type(_entries_) is Object
    6. Set _msg_ to Get(_entries_, _part_)
3. If _msg_ is **undefined**, return **undefined**.
4. If Type(_msg_) is not Object, throw a **TypeError** exception.
5. Return _msg_

### X.1.1 FormatMessageToParts ( _messageFormat_, _ctx_, _msg_ )

TODO

### X.1.1 FormatMessageToString ( _messageFormat_, _ctx_, _msg_ )

1. Let _value_ be Get(_msg_, **"value"**)
2. If IsSelectMessage(_value_), then
    1. Set _pattern_ to ? ResolveSelectMessage(_messageFormat_, _ctx_, _value_).
3. Else,
    1. Set _pattern_ to ? CreateListFromArrayLike(_value_, « Object »).
4. Let _result_ be the empty String.
5. For each element _part_ in _pattern_, do
    1. Let _fp_ be FormatMessagePart(_messageFormat_, _ctx_, _part_).
    2. Let _ps_ be FormattedMessagePartToString(_messageFormat_, _ctx_, _fp_).
    3. Set _result_ to the string-concatenation of _result_ and _ps_.
6. Return _result_.

### X.1.1 FormatMessagePart ( _messageFormat_, _ctx_, _part_ )

1. If IsMessageLiteral(_part_) is **true**, then
    1. Return _part_.
2. If IsMessageVariable(_part_) is **true**, then
    1. Return ? ResolveMessageVariable(_messageFormat_, _ctx_, _part_).
3. If IsMessageFunction(_part_) is **true**, then
    1. Return ? ResolveMessageFormatFunction(_messageFormat_, _ctx_, _part_).
4. Assert: IsMessageReference(_part_) is **true**.
5. Let _msgRef_ be ? ResolveMessageReference(_messageFormat_, _ctx_, _part_).
6. If _msgRef_\[\[Message]] is **undefined**, return `{${part.msg_path.join('.')}}`.
7. Return ? FormatMessageToString(_messageFormat_, _msgRef_\[\[Context]], _msgRef_\[\[Message]]).

### X.1.1 FormattedMessagePartToString ( _messageFormat_, _ctx_, _fmtPart_ )

TODO

### X.1.1 IsMessage ( _msg_ )

1. If Type(_msg_) is not Object, return **false**.
2. Return ? HasProperty(_msg_, **"value"**).

### X.1.1 IsMessageFunction ( _part_ )

1. If Type(_part_) is not Object, return **false**.
2. Return ? HasProperty(_part_, **"func"**).

### X.1.1 IsMessageLiteral ( _part_ )

1. If Type(_part_) is Number or String, return **true**.
2. Return **false**.

### X.1.1 IsMessageReference ( _part_ )

1. If Type(_part_) is not Object, return **false**.
2. Return ? HasProperty(_part_, **"msg_path"**).

### X.1.1 IsMessageVariable ( _part_ )

1. If Type(_part_) is not Object, return **false**.
2. Return ? HasProperty(_part_, **"var_path"**).

### X.1.1 IsSelectMessage ( _sel_ )

1. If Type(_sel_) is not Object, return **false**.
2. Return ? HasProperty(_sel_, **"select"**).

### X.1.1 MessageFormatDateTime ( _locales_, _options_, _value_ )

When the MessageFormatDateTime abstract operation is called with the arguments _locales_, _options_ and _value_, the following steps are taken: 

1. Let _options_ be ? ToDateTimeOptions(_options_, **"any"**, **"all"**).
2. Let _dateFormat_ be ? Construct(%DateTimeFormat%, « _locales_, _options_ »).
3. If Type(_value_) is String, then
    1. Let _tv_ to the result of parsing _value_ as a date, in exactly the same manner as for the `Date.parse` method.
4. Else if Type(_value_) is Object and _value_ has a \[\[DateValue]] internal slot, then
    1. Let _tv_ be _value_\[\[DateValue]].
5. Else,
    1. Let _tv_ be ? ToNumber(_value_).
6. Return ? FormatDateTime(_dateFormat_, _tv_).

### X.1.1 MessageFormatNumber ( _locales_, _options_, _value_ )

When the MessageFormatNumber abstract operation is called with the arguments _locales_, _options_ and _value_, the following steps are taken: 

1. Let _numberFormat_ be ? Construct(%NumberFormat%, « _locales_, _options_ »).
2. Let _n_ be ? ToNumber(_value_).
3. Return ? FormatNumeric(_numberFormat_, _n_).

### X.1.1 MessageSelectPlural ( _locales_, _options_, _value_ )

When the MessageSelectPlural abstract operation is called with the arguments _locales_, _options_ and _value_, the following steps are taken: 

1. Let _n_ be ? ToNumber(_value_).
2. Let _pluralRules_ be ? Construct(%PluralRules%, « _locales_, _options_ »).
3. Let _pf_ be ! ResolvePlural(_pluralRules_, _n_).
4. Return CreateArrayFromList(« _n_, _pf_ »)

### X.1.1 ResolveSelectMessage ( _messageFormat_, _ctx_, _value_ )

TODO

### X.1.1 ResolveMessagePath ( _messageFormat_, _ctx_, _path_ )

1. Let _result_ be a new empty List.
3. Let _len_ be ? ToLength(? Get(_path_, **"length"**)).
4. Let _k_ be 0.
5. Repeat, while _k_ < _len_,
    1. Let _next_ be ? Get(_path_, ! ToString(_k_)).
    2. Let _fp_ be FormatMessagePart(_messageFormat_, _ctx_, _next_)
    3. Append _fp_ as the last element of _result_.
6. Return _result_.

### X.1.1 ResolveMessageFormatFunction ( _messageFormat_, _ctx_, _part_ )

1. Let _rawArgs_ be ? Get(_part_, **"args"**).
2. Let _args_ be ? ResolveMessagePath(_messageFormat_, _ctx_, _rawArgs_).
3. Let _options_ be ? Get(_part_, **"options"**).
4. Set _options_ to ? CoerceOptionsToObject(_options_).
4. Perform ? Set(_options_, **"localeMatcher"**, _messageFormat_.\[\[LocaleMatcher]], **true**).
5. Prepend _options_ to _args_.
2. Let _locales_ be ! CreateArrayFromList(_messageFormat_.\[\[RequestedLocales]]).
6. Prepend _locales_ to _args_.
7. Let _funcName_ be ? Get(_part_, **"func"**).
8. Assert: Type(_funcName_) is String.
9. Let _func_ be _messageFormat_.\[\[Format]][\<_funcName_>].
10. If IsCallable(_func_) is **false**, throw a **RangeError** exception.
11. Return ? Call(_func_, **undefined**, _args_).

### X.1.1 ResolveMessageReference ( _messageFormat_, _ctx_, _part_ )

TODO

### X.1.1 ResolveMessageVariable ( _messageFormat_, _ctx_, _part_ )

TODO

## X.2 The Intl.MessageFormat Constructor

The MessageFormat constructor is the _%MessageFormat%_ intrinsic object and a standard built-in property of the Intl object.
Behaviour common to all [service constructor] properties is specified in [#sec-internal-slots].

### X.2.1 Intl.MessageFormat ( [ locales [ , options ] ] )

When the `Intl.MessageFormat` function is called with optional arguments _locales_ and _options_, the following steps are taken:

1. If NewTarget is **undefined**, throw a **TypeError** exception.
2. Let _messageFormat_ be ? OrdinaryCreateFromConstructor(NewTarget, "%**MessageFormat.prototype**%", « \[\[InitializedMessageFormat]], \[\[Locales]], \[\[LocaleMatcher]], \[\[Format]], \[\[Select]] »).
3. Return ? InitializeMessageFormat(_messageFormat_, _locales_, _options_).

## X.3 Properties of the Intl.MessageFormat Constructor

The Intl.MessageFormat constructor has the following properties:

### X.3.1 Intl.MessageFormat.prototype

The value of `Intl.MessageFormat.prototype` is %MessageFormat.prototype%.

This property has the attributes { \[\[Writable]]: **false**, \[\[Enumerable]]: **false**, \[\[Configurable]]: **false** }.

### X.3.1 Intl.MessageFormat.supportedLocalesOf ( locales [ , options ] )

When the `supportedLocalesOf` method is called with arguments _locales_ and _options_, the following steps are taken:

1. Let _availableLocales_ be %MessageFormat%.[[AvailableLocales]].
2. Let _requestedLocales_ be ? CanonicalizeLocaleList(_locales_).
3. Return ? SupportedLocales(_availableLocales_, _requestedLocales_, _options_).

The value of the **"length"** property of the `supportedLocalesOf` method is 1.

### X.3.1 Internal Slots

The value of the \[\[AvailableLocales]] internal slot is implementation-defined within the constraints described in [#sec-internal-slots].

The value of the \[\[RelevantExtensionKeys]] internal slot is « ».

The value of the \[\[LocaleData]] internal slot is implementation-defined within the constraints described in [#sec-internal-slots]. 

## X.4 Properties of the Intl.MessageFormat Prototype Object

The Intl.MessageFormat prototype object is itself an ordinary object. _%MessageFormat.prototype%_ is not an Intl.MessageFormat instance and does not have an \[\[InitializedMessageFormat]] internal slot or any of the other internal slots of Intl.MessageFormat instance objects. 

### X.4.1 Intl.MessageFormat.prototype.constructor

The initial value of `Intl.MessageFormat.prototype.constructor` is the intrinsic object %MessageFormat%.

### X.4.1 Intl.MessageFormat.prototype [ @@toStringTag ]

The initial value of the @@toStringTag property is the String value **"Intl.MessageFormat"**.

This property has the attributes { \[\[Writable]]: **false**, \[\[Enumerable]]: **false**, \[\[Configurable]]: **true** }.

### X.4.1 Intl.MessageFormat.prototype.format ( _resId_, _msgPath_, _scope_ )

When the `format` method is called with the arguments _resId_, _msgPath_, and _scope_, the following steps are taken:

1. Let _mf_ be the **this** value.
2. Perform ? RequireInternalSlot(_mf_, \[\[InitializedMessageFormat]]).
3. Let _ctx_ be ? GetMessageContext(_mf_, _resId_, _msgPath_, _scope_).
4. Let _msg_ be ? GetMessageEntry(_mf_, _ctx_\[\[ResourceId]], _ctx_\[\[MessagePath]]).
5. If IsMessage(_msg_) is **false**, return the empty String.
6. Return ? FormatMessageToString(_messageFormat_, _ctx_, _msg_)

### X.4.1 Intl.MessageFormat.prototype.formatToParts ( _resId_, _msgPath_, _scope_ )

When the `formatToParts` method is called with the arguments _resId_, _msgPath_, and _scope_, the following steps are taken:

1. Let _mf_ be the **this** value.
2. Perform ? RequireInternalSlot(_mf_, \[\[InitializedMessageFormat]]).
3. Let _ctx_ be ? GetMessageContext(_mf_, _resId_, _msgPath_, _scope_).
4. Let _msg_ be ? GetMessageEntry(_mf_, _ctx_\[\[ResourceId]], _ctx_\[\[MessagePath]]).
5. If IsMessage(_msg_) is **false**, return an empty Array.
6. Return ? FormatMessageToParts(_messageFormat_, _ctx_, _msg_)

### X.4.1 Intl.MessageFormat.prototype.resolvedOptions ( )

This function provides access to the locale and options computed during initialization of the object. 

## X.5 Properties of Intl.MessageFormat Instances

Intl.MessageFormat instances are ordinary objects that inherit properties from %MessageFormat.prototype%.

Intl.MessageFormat instances have an \[\[InitializedMessageFormat]] internal slot.

Intl.MessageFormat instances also have several internal slots that are computed by the constructor: 

- \[\[RequestedLocales]] is a List that contains structurally valid ([#sec-isstructurallyvalidlanguagetag]) and canonicalized ([#sec-canonicalizeunicodelocaleid]) Unicode BCP 47 locale identifiers identifying the requested locales, in preferential order.
- \[\[LocaleMatcher]] is one of the String values **"lookup"**, **"best fit"**, identifying the locale matcher to use 
- \[\[Format]] is a Record of formatter functions that may be used within messages.
- \[\[Select]] is a Record of case selector functions that may be used within selector messages.
