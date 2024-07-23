**Author:** <br>
[Rashmi Rao](https://github.com/rashmijrao), [John Cox](https://github.com/johncox-google), [Salman Malik](https://github.com/salmanmlk), Google Privacy Sandbox

# Protected App Signals egress in Bidding and Auction Services

Protected App Signals provides ad-techs a way to egress data from inside the privacy boundary to their own servers for model training. See the [Protected App Signals Explainer](https://developers.google.com/privacy-sandbox/relevance/protected-audience/android/protected-app-signals), particularly the [Reporting](https://developers.google.com/privacy-sandbox/relevance/protected-audience/android/protected-app-signals#reporting) section, for background. Refer design for Protected App Signals with B&A [here](https://github.com/privacysandbox/protected-auction-services-docs/blob/main/bidding_auction_services_protected_app_signals.md).

This document describes the following:

1. The set of feature types available to egress information to adtech servers for model training for Protected App Signals.

2. The wire format of each feature.

3. The wire format of the `egressPayload` itself.

The wire representation of the `egressPayload` will be noised per feature type. More details will be provided in a future explainer update.

Based on the information in this document, adtechs can write parsers to prepare the `egressPayload` and transform the values it contains into features for use in their model training systems. 

# Feature types

We support two kinds of feature types: primitives, which can contain a single feature value; and collections, which can contain multiple primitives.

**Nullability**

_(Nullability will be a fast-follow feature; it will not be available immediately upon the release of egress vector. Please see this addendum describing a schema modification so your de-serializer can be prepared for nullability support.)_

Every feature type can optionally be made nullable, in which case the value `null` can be represented. By default features are _not_ nullable. A null value for a type is indicated by adding a signaling bit, 0 (for null) or 1 (for non-null), to the least-significant bit of the serialized representation. Consequently, the wire representation for a feature with `nullable = true` is always 1 bit larger than with `nullable = false`. 

The contents of collections can be nullable and are handled as above. Collections themselves can also be nullable; handled the same way: a null value's wire representation is 0s for all value bits, with the 0 signaling bit.

Because values are _not_ nullable by default, values mentioned in the examples below are not nullable unless stated.

## Primitive types

These types represent single values: booleans, unsigned integers, and signed integers

### `boolean-feature-type`

This represents a single boolean value.

Expected value: `true` or `false`, or optionally `null`

Parameters:

*   `nullable`: boolean indicating whether the `boolean-feature-type` can represent a null state or not. Defaults to `false`.

**Wire format**

Non-nullable:

0 (`false`) or 1 (`true`)

Nullable:

01 (`false`) or 11 (`true`), or 00 (`null`)

### `unsigned-integer-feature-type`

This represents a single non-negative integer value.

Parameters:

*   `size`: unsigned integer indicating the number of bits based on the range of values.

      * Expected value:  non-negative integer value in the range [0,2^<sup>size</sup>-1]

      * For example, consider an `unsigned-integer-feature-type` of `size` = `3`. It can have a value: ` [0, 7]` and will occupy 3 bits on the wire (or four bits on the wire if nullable, see examples below).
*   `nullable`: boolean indicating whether the `unsigned-integer-feature-type` can represent a null state or not. Defaults to `false`.

**Wire format**

Binary representation of the unsigned integer. The integer will be converted to wire format following little endian byte order. If nullable, will be prefixed by the indicator byte.

Examples:

*Non-nullable*:

If `value` = `5` and `size `= `3`, wire format will be `101`

*Nullable*:

If `value` = `5` and `size `= `3`, wire format will be `1011`.

If `value` = `0` and `size` = `3`, wire format will be `0001`

If `value` = `null` and `size` = `3`, wire format will be `0000`

### `signed-integer-feature-type`

This type can be used to represent a single positive or negative integer value. 

Parameters:

*   `size`: unsigned integer indicating the number of bits based on the range of values.

    * Expected value:  integer value in the range [-2^<sup>(size-1)</sup>,2^<sup>(size-1)</sup>-1]

    * For example, consider a `signed-integer-feature-type` of `size` = `4`. It can have a value `[-8, 7]`, and will occupy 4 bits on the wire (or 5 bits on the wire if nullable, see examples below).

*   `nullable`: boolean indicating whether the `signed-integer-feature-type` can represent a null state or not. Defaults to `false`.

**Wire format**

2’s complement representation of the signed integer. The integer will be converted to wire format following little endian byte order.

Examples:

*Non-nullable*:

If `value` = `-3` and `size`=`4`, wire format will be `1101`

*Nullable*:

If `value` = `-3` and `size`=`4`, wire format will be `11011`

If `value` = `0` and `size` = `4`, wire format will be `00001`

If `value` = `null` and `size` = `4`, wire format will be `00000`


## Collection types

These feature types represent a collection of homogeneous or heterogeneous values.

The wire representation of the values will be in the right-to-left order.

### `bucket-feature-type`

This type can be used to represent an ordered list of `boolean-feature-type` values.

Expected value: list of boolean (`true` or `false`, or optionally `null`) values, or optionally `null`

**Parameters**

*   `allow-multiple`: indicates whether the bucket can contain multiple `true` values.
*   `size`: number of values in the bucket.
*   `nullable`: boolean indicating whether the `bucket-feature-type` as a whole can represent a null state or not. Defaults to `false`.

**Wire format**

Sequential bit representation of `boolean-feature-type` values.

Examples:

*Non-nullable collection*:

Consider a `bucket-feature-type` of `size` = `4`. If the values are [`true`, `false`, `true`, `false`] this will occupy `4` bits on the wire. Wire format will be `0101`

Again for `size` = `4`, now consider the values all being `nullable = true`. If the values are [`true`, `false`, `true`, `null`] this will occupy `8` bits on the wire. Wire format will be `00 11 01 11` (spaces for readability).

*Nullable collection*

Consider a `bucket-feature-type` of `size` = `4`. If the values are [`true`, `false`, `true`, `false`] this will occupy `5` bits on the wire. Wire format will be `01011`

### `histogram-feature-type`

This type can be used to represent an ordered, heterogeneous list of `unsigned-integer-feature-type` and `signed-integer-feature-type` values. 

**Parameters**

*   `size`: unsigned integer indicating the number of fixed size values in the histogram.

Expected values: list of `unsigned-integer-feature-type` and `signed-integer-feature-type` values

**Wire format**

In general the wire format for the histogram is the wire format of each contained value, right-to-left.

Examples:

*Non-nullable histogram*:

If the histogram contains `2` elements, the first of which is an unsigned 3-bit integer with the value `5`, and the second is a signed `4`-bit integer with the value `-3`, then the wire format would be `1101 101` (spaces for readability).

Consider the same example, except the second value is now nullable. Now the wire format requires eight bits instead of seven, and becomes `11011 101`.

Consider the same example again, except now the second value is nullable and indeed `null`. The wire format still requires eight bits and is `00000 101`.

*Nullable histogram*: 

Consider the original example, except the histogram is now itself nullable. Now the representation requires eight bits, becoming `1101 101 1`.

Now consider if both the histogram itself is nullable, and the second value is nullable and indeed `null`. The wire format now requires nine bits and is `00000 101 1`.

# Payload wire format

The definition of the wire format of a payload is called its _protocol_. Below we describe the first wire format, or protocol version `1`.  The protocol version included in the payload will be set by the platform.

A payload is made up of two parts: a **header** containing metadata information used for serialization and deserialization; and a **body** containing serialized feature values.

## Header

The header itself has two parts: 

1. **Protocol version**: unsigned `5`-bit int indicating the version of the wire format specification used to encode the payload.

2. **Schema version**: unsigned `3`-bit int. Version identifier for the schema that defines the payload.

The wire format of the header is the protocol version, then the schema version, right-to-left. For example, if the protocol version is `1` (`00001` on the wire)  and the schema version is `2` (`010` on the wire), the header will be `01000001`.

## Body

The body contains serialized feature values, with values as defined in each feature type above. The order of the features is the same as their order in the provided schema for the payload, right-to-left.

The body is `0`-padded. Details of the padding are slightly different for `egressPayload` and `temporaryUnlimitedEgressPayload`:

1. `egressPayload` is first `0`-padded to its maximum size in bits, then `0`-padded to the nearest byte.
2. `temporaryUnlimitedEgressPayload` is `0`-padded to the nearest byte.

## Example 

*   Protocol version : `1` 
*   Example schema version : `2` 
*   Example max wire size for `egressPayload`: `20` bits

Consider this example schema, feature values and corresponding wire format for each feature type specified the schema:

<table>
  <tr>
   <td colspan="2"><strong>Feature type</strong>
<p>
<strong>(Defined in the schema)</strong>
   </td>
   <td><strong>Feature type in collection</strong>
<p>
<strong>(Defined in the schema)</strong>
   </td>
   <td><strong>Parameters </strong>
<p>
<strong>(Defined in the schema)</strong>
   </td>
   <td><strong>Corresponding value in Json</strong>
   </td>
   <td><strong>Wire representation</strong>
   </td>
  </tr>
  <tr>
   <td rowspan="2" colspan="2"><code>histogram-feature-type</code> with <code>size</code> = <code>2</code>
   </td>
   <td><code>unsigned-int-feature-type</code>
   </td>
   <td><code>size</code> = <code>3</code>
   </td>
   <td><code>5</code>
   </td>
   <td><code>101</code>
   </td>
  </tr>
  <tr>
   <td><code>signed-int-feature-type</code>
   </td>
   <td><code>size</code> = <code>4</code>
   </td>
   <td><code>-3</code>
   </td>
   <td><code>1101</code>
   </td>
  </tr>
  <tr>
   <td colspan="2"><code>boolean-feature-type</code>
   </td>
   <td>
   </td>
   <td>
   </td>
   <td><code>false</code>
   </td>
   <td><code>0</code>
   </td>
  </tr>
  <tr>
   <td colspan="2"><code>bucket-feature-type</code> 
   </td>
   <td><code>boolean-feature-type</code>
   </td>
   <td><code>size</code> = <code>4</code>, <code>allow-multiple</code> = <code>true</code>,
   <code>nullable</code> = <code>true</code>
   </td>
   <td><code>[true, false, true, false]</code>
   </td>
   <td><code>01011</code>
   </td>
  </tr>
  <tr>
   <td colspan="2"><code>boolean-feature-type</code>
   </td>
   <td>
   </td>
   <td>
    <code>nullable</code> = <code>true</code>
   </td>
   <td><code>null</code>
   </td>
   <td><code>00</code>
   </td>
  </tr>
</table>


Wire format of the feature values would be: 
<img src="images/feature-values-wire-format.png" width="80%" alt="Feature values in wire format">

#### Wire format for <code>temporaryUnlimitedEgressPayload</code>:</strong>

Wire format of the feature values + padding would be: 

<img src="images/temporary-unlimited-egress-with-padding.png" width="80%" alt="Wire format for temporaryUnlimitedEgressPayload with padding">

Wire format of the feature values + padding + header would be:

<img src="images/temporary-unlimited-egress-payload.png" width="80%" alt="Wire format for temporaryUnlimitedEgressPayload with padding and header">

#### Wire format for <code>egressPayload</code>:</strong>

Wire format of the feature values + padding would be: 

<img src="images/egress-payload-with-padding.png" width="80%" alt="Wire format for egressPayload with padding">

Wire format of the feature values + padding + header would be:

<img src="images/egress-payload.png" width="80%" alt="Wire format for egressPayload with padding">

## Addendum: Workaround for implementing nullability before official support

While nullability will not be supported immediately upon the release of egress vector support, AdTechs can write their schemas in such a way that the serialized representation of the egress vector is identical before and after nullability is supported.

Recall that a null value for a type is indicated by adding a signaling bit to the least-significant bit of the serialized representation. AdTechs can simply add a `boolean-feature-type` before each value which they intend to make nullable, and set it to indicate whether the following value is null: 0 (for null) or 1 (for non-null).

Because this approach is identical to the way nullability support will be implemented, it allows AdTechs to write de-serialziation code now. Upon the release of nullability support, AdTechs will remove the boolean nullability flags and mark the values to which they correspond as nullable in the schema, and update their code for building the egress vector accordingly.

Consider the example schema from above, modified to show nullability flags <b>in bold</b> before nullability support:

<table>
  <tr>
   <td colspan="2"><strong>Feature type</strong>
<p>
<strong>(Defined in the schema)</strong>
   </td>
   <td><strong>Feature type in collection</strong>
<p>
<strong>(Defined in the schema)</strong>
   </td>
   <td><strong>Parameters </strong>
<p>
<strong>(Defined in the schema)</strong>
   </td>
   <td><strong>Corresponding value in Json</strong>
   </td>
   <td><strong>Wire representation</strong>
   </td>
  </tr>
  <tr>
   <td rowspan="2" colspan="2"><code>histogram-feature-type</code> with <code>size</code> = <code>2</code>
   </td>
   <td><code>unsigned-int-feature-type</code>
   </td>
   <td><code>size</code> = <code>3</code>
   </td>
   <td><code>5</code>
   </td>
   <td><code>101</code>
   </td>
  </tr>
  <tr>
   <td><code>signed-int-feature-type</code>
   </td>
   <td><code>size</code> = <code>4</code>
   </td>
   <td><code>-3</code>
   </td>
   <td><code>1101</code>
   </td>
  </tr>
  <tr>
   <td colspan="2"><code>boolean-feature-type</code>
   </td>
   <td>
   </td>
   <td>
   </td>
   <td><code>false</code>
   </td>
   <td><code>0</code>
   </td>
  </tr>
  <tr>
   <td colspan="2"><code><b>boolean-feature-type</b></code>
   </td>
   <td>
   </td>
   <td>
   </td>
   <td><code><b>true</b></code>
   </td>
   <td><code><b>1</b></code>
   </td>
  </tr>
  <tr>
   <td colspan="2"><code>bucket-feature-type</code> 
   </td>
   <td><code>boolean-feature-type</code>
   </td>
   <td><code>size</code> = <code>4</code>, <code>allow-multiple</code> = <code>true</code>
   </td>
   <td><code>[true, false, true, false]</code>
   </td>
   <td><code>0101</code>
   </td>
  </tr>
  <tr>
   <td colspan="2"><code><b>boolean-feature-type</b></code>
   </td>
   <td>
   </td>
   <td>
   </td>
   <td><code><b>false</b></code>
   </td>
   <td><code><b>0</b></code>
   </td>
  </tr>
  <tr>
   <td colspan="2"><code>boolean-feature-type</code>
   </td>
   <td>
   </td>
   <td>
   </td>
   <td><code>false</code>
   </td>
   <td><code>0</code>
   </td>
  </tr>
</table>

The wire format displayed in the diagram above still applies - which is why this workaround allows de-serialization code to be written before nullability support is released.
