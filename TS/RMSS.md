# AV1 Codec ISO Media File Format Binding

v1.0.0, 7 September 2018

**This version:**
https://aomediacodec.github.io/av1-isobmff/

**Issue Tracking:**
GitHub

**Editors:**
Cyril Concolato (Netflix)
Tom Finegan (Google)

Copyright 2018, The Alliance for Open Media

Licensing information is available at http://aomedia.org/license/

The MATERIALS ARE PROVIDED “AS IS.” The Alliance for Open Media, its members, and itscontributors expressly disclaim any warranties (express, implied, or otherwise), including implied war-ranties of merchantability, non-infringement, fitness for a particular purpose, or title, related to the ma-terials. The entire risk as to implementing or otherwise using the materials is assumed by the imple-menter and user. IN NO EVENT WILL THE ALLIANCE FOR OPEN MEDIA, ITS MEMBERS, ORCONTRIBUTORS BE LIABLE TO ANY OTHER PARTY FOR LOST PROFITS OR ANY FORMOF INDIRECT, SPECIAL, INCIDENTAL, OR CONSEQUENTIAL DAMAGES OF ANY CHAR-ACTER FROM ANY CAUSES OF ACTION OF ANY KIND WITH RESPECT TO THIS DELIV-ERABLE OR ITS GOVERNING AGREEMENT, WHETHER BASED ON BREACH OF CON-TRACT, TORT (INCLUDING NEGLIGENCE), OR OTHERWISE, AND WHETHER OR NOT THEOTHER MEMBER HAS BEEN ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
 
**Abstract** 
This document specifies the storage format for [AV1] bitstreams in [ISOBMFF] tracks as well as in[CMAF] files. 
 
## Bitstream features overview 

An *_AV1 bitstream_* is composed of a sequence of *OBUs*, grouped into *Temporal Units*.

OBUs are made of a 1 or 2 bytes header, identifying in particular the type of OBU, followed by an op-tional length field and by an optional payload, whose presence and content depend on the OBU type.Depending on its type, an OBU can carry configuration information, metadata, or coded video data.

```
NOTE: Tile List OBUs defined in the [AV1] specification are not supported in the current versionof this specification. 
A future version of the specification may do so.
```
Temporal Units are processed by a decoder in the order given by the bitstream. Each Temporal Unit isassociated with a presentation time. Some Temporal Units may contain multiple frames to be decodedbut only one is presented.

## Basic Encapsulation Scheme

This section describes the basic data structures used to signal encapsulation of *AV1 bitstreams* in [ISOBMFF] containers.

## General Requirements & Brands

A file conformant to this specification satisfies the following:

It SHALL conform to the normative requirements of [ISOBMFF]
It SHALL have the *‘av01’* brand among the compatible brands array of the FileTypeBox
It SHALL contain at least one track using an *AV1SampleEntry*
It SHOULD indicate a structural ISOBMFF brand among the compatible brands array of the File-TypeBox, such as *‘iso6’*
It MAY indicate CMAF brands as specified in *§3 CMAF AV1 track format*
It MAY indicate other brands not specified in this document provided that the associated require-ments do not conflict with those given in this specification

Parsers SHALL support the structures required by the *‘iso6’* brand and MAY support structures re-quired by further ISOBMFF structural brands.

## AV1 Sample Entry

### Definitions

```
Sample Entry Type: **av01**
Container:         Sample Description Box ('stsd')
Mandatory:         Yes
Quantity:          One or more.
```
### Description
The **_AV1SampleEntry_** sample entry identifies that the track contains *AV1 Samples*, and uses an *AV1-CodecConfigurationBox*.

### Syntax
```
class AV1SampleEntry extends VisualSampleEntry('av01') {  
AV1CodecConfigurationBox config;
}
```
### Semantics

The **_width_** and **_height_** fields of the *VisualSampleEntry* SHALL equal the values of *max_frame_width_minus_1* + 1 and *max_frame_height_minus_1* + 1 of the *Sequence Header OBU* applying to the samples associated with this sample entry.

The width and height in the TrackHeaderBox SHOULD equal, respectively, the maximum Render-Width, called MaxRenderWidth, and the maximum RenderHeight, called MaxRenderHeight, of all theframes associated with this sample entry. Additionally, if MaxRenderWidth and MaxRenderHeightvalues do not equal respectively the *max_frame_width_minus_1* + 1 and *max_frame_height_minus_1* + 1 values of the *Sequence Header OBU*, a PixelAspectRatioBox box SHALL be present in the sampleentry and set such that

```
hSpacing / vSpacing = MaxRenderWidth * (max_frame_height_minus_1 + 1) / 
((max_frame_width_minus_1 + 1) * MaxRenderHeig
```
The **_compressorname_** field of the *VisualSampleEntry* is an informative name. It is formatted in a fixed32-byte field, with the first byte set to the number of bytes to be displayed, followed by that number ofbytes of displayable data, followed by padding to complete 32 bytes total (including the size byte).The value "\012AOM Coding" is RECOMMENDED; the first byte is a count of the remaining bytes, here represented by \012, which (being octal 12) is decimal 10, the number of bytes in the rest of thestring.

```
NOTE: Parsers may ignore the value of the compressorname field. 
It is specified in this documentsimply for legacy and backwards compatibility reasons.
```

The **_config_** field SHALL contain an*AV1CodecConfigurationBox* that applies to the samples associat-ed with this sample entry

```
NOTE: Multiple instances of *AV1SampleEntry* may be required when the track contains samples
requiring a *AV1CodecConfigurationBox* with different characteristics.
```

Optional boxes not specifically mentioned here can be present, in particular those indicated in the defi-nition of the *VisualSampleEntry* in [ISOBMFF].

## AV1 Codec Configuration Box
### Definition

```
Box Type:  av1C
Container: AV1 Sample Entry ('av01')
Mandatory: Yes
Quantity:  Exactly OneThe
```

### Description

The **_AV1CodecConfigurationBox_** contains decoder configuration information that SHALL be validfor every sample that references the sample entry.

### Syntax
```
class AV1CodecConfigurationBox extends Box('av1C'){
  AV1CodecConfigurationRecord av1Config;
}

aligned (8) class AV1CodecConfigurationRecord {
 unsigned int (1) marker = 1;
 unsigned int (7) version = 1;
 unsigned int (3) seq_profile;
 unsigned int (5) seq_level_idx_0;
 unsigned int (1) seq_tier_0;
 unsigned int (1) high_bitdepth;
 unsigned int (1) twelve_bit;
 unsigned int (1) monochrome;
 unsigned int (1) chroma_subsampling_x;
 unsigned int (1) chroma_subsampling_y;
 unsigned int (2) chroma_sample_position;
 unsigned int (3) reserved = 0;

 unsigned int (1) initial_presentation_delay_present;
 if (initial_presentation_delay_present) {
 unsigned int (4) initial_presentation_delay_minus_one;  
 } else {
  unsigned int (4) reserved = 0;
 }

  unsigned int (8)[] configOBUs;
 }
```

### Semantics

The **_marker_** field SHALL be set to 1.

```
NOTE: The marker bit ensures that the bit pattern of the first byte of the AV1CodecConfiguration-
Record cannot be mistaken for an OBU Header byte.
```

The **_version_** field indicates the version of the AV1CodecConfigurationRecord. The value SHALL beset to 1 for AV1CodecConfigurationRecord.

The **_seq_profile_** field indicates the AV1 profile and SHALL be equal to the seq_profile value from the *Sequence Header OBU*.

The **_seq_level_idx_0_** field indicates the value of seq_level_idx[0] found in the Sequence Header OBU and SHALL be equal to the value of seq_level_idx[0] in the *Sequence Header OBU*.

The **_seq_tier_0_** field indicates the value of seq_tier[0] found in the *Sequence Header OBU* and SHALLbe equal to the value of seq_tier[0] in the *Sequence Header OBU*.

The **_high_bitdepth_** field indicates the value of the *high_bitdepth* flag from the *Sequence Header OBU*.

The **_twelve_bit_** field indicates the value of the *twelve_bit* flag from the *Sequence Header OBU*.

The **_monochrome_** field indicates the value of the *mono_chrome* flag from the *Sequence Header OBU*.

The **_chroma_subsampling_x_** field indicates the *subsampling_x* value from the *Sequence Header OBU*.

The **_chroma_subsampling_y_** field indicates the *subsampling_y* value from the *Sequence Header OBU*.

The **_chroma_sample_position_** field indicates the *chroma_sample_position* value from the *SequenceHeader OBU*.

The **_initial_presentation_delay_present_** field indicates the presence of the initial_presentation_delay_minus_one field.

The **_initial_presentation_delay_minus_one_** field indicates the number of samples (minus one) that need to be decoded prior to starting the presentation of the first sample associated with this sample entry in order to guarantee that each sample will be decoded prior to its presentation time under the constraints of the first level value indicated by *seq_level_idx in* the *Sequence Header OBU* (in the config-OBUs field or in the associated samples). More precisely, the following procedure SHALL not return any error:

- construct a hypothetical bitstream consisting of the OBUs carried in the sample entry followed by the OBUs carried in all the samples referring to that sample entry,
- set the first *initial_display_delay_minus_1* field of each *Sequence Header OBU* to the number offrames minus one contained in the first *initial_presentation_delay_minus_one* + 1 samples,
- set the *frame_presentation_time* field of the frame header of each presentable frame such that it matches the presentation time difference between the sample carrying this frame and the previous sample (if it exists, 0 otherwise),
- apply the decoder model specified in [AV1] to this hypothetical bitstream using the first operating point. If **_buffer_removal_time_** information is present in bitstream for this operating point, the decoding schedule mode SHALL be applied, otherwise the resource availability mode SHALL be applied.

```
NOTE: With the above procedure, when smooth presentation can be guaranteed after decoding the
first sample, initial_presentation_delay_minus_one is 0.
```
```
NOTE: Because the above procedure considers all OBUs in all samples associated with a sample
entry, if these OBUS form multiple coded video sequences which would have different values of
initial_presentation_delay_minus_one if considered separately, the sample entry would
signal the larger value.
```
```
EXAMPLE 1
The difference between initial_presentation_delay_minus_one and initial_display_delay_minus_1 can be illustrated by considering the following example:

a b c d e f g h

where letters correspond to frames. Assume that initial_display_delay_minus_1 is 2, i.e. that once frame c has been decoded, all other frames in the bitstream can be presented on time. If those frames were grouped into temporal units and samples as follows:

[a] [b c d] [e] [f] [g] [h]

initial_presentation_delay_minus_one would be 1 because it takes presentation of 2 samples to ensure that c is decoded. But if the frames were grouped as follows:

[a] [b] [c] [d e f] [g] [h]

initial_presentation_delay_minus_one would be 2 because it takes presentation of 3 samples to ensure that c is decoded.
```
The **_configOBUs_** field contains zero or more OBUs. Any OBU may be present provided that the following procedures produce compliant AV1 bitstreams:

- From any sync sample, an *AV1 bitstream is formed by first outputting* the OBUs contained in the AV1CodecConfigurationBox and then by outputing all OBUs in the samples themselves, in order, starting from the sync sample.
- From any sample marked with the *AV1ForwardKeyFrameSampleGroupEntry*, an AV1 bitstream is formed by first outputting the OBUs contained in the *AV1CodecConfigurationBox* and then by outputing all OBUs in the sample itself, then by outputting all OBUs in the samples, in order, starting from the sample at the distance indicated by the sample group.

Additionally, the configOBUs field SHALL contain at most one *Sequence Header OBU* and if present, it SHALL be the first OBU.

```
NOTE: The configOBUs field is expected to contain only one *Sequence Header OBU* and zero or more *Metadata OBUs* when applicable to all the associated samples.
```

OBUs stored in the configOBUs field follow the *open_bitstream_unit Low Overhead Bitstream Format*
syntax as specified in [AV1]. The flag *obu_has_size_field* SHALL be set to 1, indicating that the
size of the OBU payload follows the header, and that it is coded using LEB128.

When a *Sequence Header OBU* is contained within the configOBUs of the AV1CodecConfiguration-
Record, the values present in the *Sequence Header OBU* contained within configOBUs SHALL match
the values of the AV1CodecConfigurationRecord.

The presentation times of AV1 samples are given by the ISOBMFF structures. The *timing_info_present_
flag* in the *Sequence Header OBU* (in the configOBUs field or in the associated samples)
SHOULD be set to 0. If set to 1, the *timing_info* structure of the *Sequence Header OBU*, the *frame_presentation_
time* and *buffer_removal_time* fields of the *Frame Header OBUs*, if present, SHALL be
ignored for the purpose of timed processing of the ISOBMFF file.

The sample entry SHOULD contain a *‘colr’* box with a colour_type set to *‘nclx’*. If present, the values
of colour_primaries, transfer_characteristics, and matrix_coefficients SHALL match the values given
in the *Sequence Header* OBU (in the configOBUs field or in the associated samples) if the color_description_
present_flag is set to 1. Similarly, the full_range_flag in the *‘colr’* box shall match the color_
range flag in the *Sequence Header OBU8. When configOBUs does not contain a *Sequence Header
OBU*, this box with colour_type set to *‘nclx’* SHALL be present.

The CleanApertureBox *‘clap’* SHOULD not be present.

For sample entries corresponding to HDR content, the MasteringDisplayColourVolumeBox *‘mdcv’*
and ContentLightLevelBox *‘clli’* SHOULD be present, and their values SHALL match the values of
contained in the *Metadata OBUs* of type METADATA_TYPE_HDR_CLL and METADATA_TYPE-
_HDR_MDCV, if present (in the configOBUs or in the samples).

```
NOTE: The MasteringDisplayColourVolumeBox ‘mdcv’ and ContentLightLevelBox ‘clli’ have
identical syntax to the SMPTE2086MasteringDisplayMetadataBox ‘SmDm’ and ContentLight-
LevelBox ‘CoLL’, except that they are of type Box and not FullBox. However, the semantics of
the MasteringDisplayColourVolumeBox and SMPTE2086MasteringDisplayMetadataBox have important
differences and should not be confused.
```

Additional boxes may be provided at the end of the *VisualSampleEntry* as permitted by ISOBMFF,
that may represent redundant or similar information to the one provided in some OBUs contained in
the *AV1CodecConfigurationBox*. If the box definition does not indicate that its information overrides
the OBU information, in case of conflict, the OBU information should be considered authoritative.

## AV1 Sample Format

For tracks using the *AV1SampleEntry*, an **_AV1 Sample_** has the following constraints:

- the sample data SHALL be a sequence of *OBUs* forming a *Temporal Unit*,
- each OBU SHALL follow the *open_bitstream_unit Low Overhead Bitstream Format* syntax as
specified in [AV1]. Each OBU SHALL have the *obu_has_size_field* set to 1 except for the last
OBU in the sample, for which obu_has_size_field MAY be set to 0, in which case it is assumed to
fill the remainder of the sample,

```
NOTE: When extracting OBUs from an ISOBMFF file, and depending on the capabilities of the
decoder processing these OBUs, ISOBMFF parsers MAY need to either: set the obu_has_size_-
field to 1 for some OBUs if not already set, add the size field in this case, and add Temporal Delimiter
OBU; or use the length-delimited bitstream format as defined in Annex B of AV1. If encryption
is used, similar operations MAY have to be done before or after decryption depending on
the demuxer/decryptor/decoder architecture.
```

- OBU trailing bits SHOULD be limited to byte alignment and SHOULD not be used for padding,
- OBUs of type OBU_TEMPORAL_DELIMITER, OBU_PADDING, or OBU_REDUNDAN-T_FRAME_HEADER SHOULD NOT be used.
- OBUs of type OBU_TILE_LIST SHALL NOT be used.

If an AV1 Sample is signaled as a sync sample (in the SyncSampleBox or by setting sample_is_non_-sync_sample to 0), it SHALL be a Random Access Point as defined in [AV1], i.e. satisfy the followingconstraints:

- Its first frame is a *Key Frame* that has *show_frame8 flag set to 1,
- It contains a *Sequence Header OBU* before the first *Frame Header OBU*.

```
NOTE: Within this definition, a sync sample may contain additional frames that are not KeyFrames. The fact that none of them is the first frame in the temporal unit ensures that they aredecodable.
```

```
NOTE: Other types of OBUs such as *metadata OBUs* could be present before the *Sequence Header OBU*.
```

*Intra-only frames* SHOULD be signaled using the sample_depends_on flag set to 2.

*Delayed Random Access Points* SHOULD be signaled using sample groups and the *AV1ForwardKey-FrameSampleGroupEntry*.

*Switch Frames* SHOULD be signaled using sample groups and the *AV1SwitchFrameSampleGroupEn-try*.

Additionally, if a file contains multiple tracks that are alternative representations of the same content,in particular using *Switch Frames*, those tracks SHOULD be marked as belonging to the same alter-nate group and should use a track selection box with an appropriate attribute (e.g. *‘bitr’*).

In tracks using the *AV1SampleEntry*, the ‘ctts’ box and composition offsets in movie fragmentsSHALL NOT be used. Similarly, the is_leading flag, if used, SHALL be set to 0 or 2.

When a temporal unit contains more than one frame, the sample corresponding to that temporal unitMAY be marked using the *AV1MultiFrameSampleGroupEntry*.

*Metadata OBUs* may be carried in sample data. In this case, the *AV1MetadataSampleGroupEntry* SHOULD be used. If the *metadata OBUs* are static for the entire set of samples associated with a giv-en sample description entry, they SHOULD also be in the OBU array in the sample description entry.

Unless explicitely stated, the grouping_type_parameter is not defined for the SampleToGroupBox withgrouping types defined in this specification.

## AV1 Forward Key Frame sample group entry

### Definition

```
Group Type: av1f
Container:  Sample Group Description Box ('sgpd')
Mandatory:  No
Quantity:   Zero or more.
```

#### Description

The **_AV1ForwardKeyFrameSampleGroupEntry_** documents samples that contain a *Delayed Random Access Point* that are followed at a given distance in the bitstream by a *Key Frame Dependent Recov-ery Point*.

### Syntax
```
class AV1ForwardKeyFrameSampleGroupEntry extends VisualSampleGroupEntry('av1  
unsigned int(8) fwd_distance;
}
```
### Semantics

The **_fwd_distance_** field indicates the number of samples between this sample and the next sample con-taining the associated *Key Frame Dependent Recovery Point*. 0 means the next sample.

## AV1 Multi-Frame sample group entry

### Definition
```
Group Type: av1m
Container:  Sample Group Description Box ('sgpd')
Mandatory:  NoQuantity:   
Zero or more.
```

### Description

The **_AV1MultiFrameSampleGroupEntry_** documents samples that contain multiple frames.

### Syntax

```
class AV1MultiFrameSampleGroupEntry extends VisualSampleGroupEntry('av1m') {
}
```

## AV1 Switch Frame sample group entry

### Definition

```
Group Type: av1s
Container:  Sample Group Description Box ('sgpd')
Mandatory:  NoQuantity:   
Zero or more
```

### Description

The **_AV1SwitchFrameSampleGroupEntry_** documents samples that start with a *Switch Frame*.

### Syntax

```
class AV1SwitchFrameSampleGroupEntry extends VisualSampleGroupEntry('av1s') 
}
```

### Description

The AV1MetadataSampleGroupEntry documents samples that contain *metadata OBUs*. Thegrouping_type_parameter can be used to identify samples containing *metadata OBUs* of a giventype. If no grouping_type_parameter is provided, the sample group entry identifies samples con-taining *metadata OBUs* for which the metadata_type is unknown.

 ### Syntax
```
class AV1MetadataSampleGroupEntry extends VisualSampleGroupEntry('av1M') {
}
```
For this sample group entry, the grouping_type_parameter syntax is as follows
```
{   
   unsigned int (8) metadata_type;   
   unsigned int (24) metadata_specific_parameters;
}
```

### Semantics

**_metadata_type_** is a 8-bit field whose value is the value of the metadata_type field defined in [AV1],when it is equal or lower than 255. metadata_type values above 255 are not supported by this samplegroup.

**_metadata_specific_parameters provides_** an additional part of the grouping_type_parameter,which MAY be used to distinguish different sample groups having the same metadata_type but dif-ferent sub-parameters. In this version of the specification, metadata_specific_parameters is only defined when metadata_type is set to METADATA_TYPE_ITUT_T35 in which case its value SHALL be set to the first 24 bits of the metadata_itut_t35 structure. For other types of metadata,its value SHOULD be set to 0. In all cases, when the grouping_type_parameter is used, readers processing sample groups SHALL use the entire value of grouping_type_parameter.

# CMAF AV1 track format

[CMAF] defines structural constraints on ISOBMFF files additional to [ISOBMFF] for the purpose of,for example, adaptive streaming or for protected files. Conformance to these structural constraints issignaled by the presence of the brand cmfc in the FileTypeBox.

If a CMAF Video Track uses the brand av01, it is called a CMAF AV1 Track and the following con-straints, defining the CMAF Media Profile for AV1, apply:

- it SHALL use an *AV1SampleEntry*
- it MAY use multiple sample entries, and in that case the following values SHALL not change inthe track:
  - seq_profile
  - still_picture
  - the first value of seq_level_idx
  - the first value of seq_tier
  - color_config
  - initial_presentation_delay_minus_one

When protected, *CMAF AV1 Tracks* SHALL use the signaling defined in [CMAF], which in turn re-lies on [CENC], with the provisions specified in *§4 Common Encryption*.

# Common Encryption

*CMAF AV1 Tracks* and non-segmented AV1 files MAY be protected. If protected, they SHALL con-form to [CENC]. Only the *cenc* and *cbcs* protection schemes are supported.

When the protected scheme *cenc* is used, samples SHALL be protected using subsample encryptionand SHALL NOT use pattern encryption.

When the protected scheme *cbcs* is used, samples SHALL be protected using subsample encryption and SHALL use pattern encryption. [CENC] recommends that the Pattern Block length be 10, i.e.crypt_byte_block + skip_byte_block = 10. All combinations such that this is true SHALLbe supported, including crypt_byte_block = 10 and skip_byte_block = 0.

## General Subsample Encryption constraints§

Within protected samples, the following constraints apply:

- Protected samples SHALL be exactly spanned by one or more contiguous subsamples.
  - An OBU MAY be spanned by one or more subsamples, especially when it has multipleranges of protected data. However, as recommended in [CENC], writers SHOULD reducethe number of subsamples as possible. This can be achieved by using a subsample that spansmultiple consecutive unprotected OBUs as well as the first unprotected and protected partsof the following protected OBU, if such protected OBU exists.
  - A large unprotected OBU whose data size is larger than the maximum size of a single *Bytes-Of ClearData* field MAY be spanned by multiple subsamples with zero size *BytesOfPro-tectedData*.

- All *OBU Headers* and associated *obu_size* fields SHALL be unprotected.
- *Temporal Delimiter OBUs, Sequence Header OBUs*, and *Frame Header OBUs* (including withina *Frame OBU*), *Redundant Frame Header OBUs* and *Padding OBUs* SHALL be unprotected.
- *Metadata OBUs* MAY be protected.
- *Tile Group OBUs* and *Frame OBUs* SHALL be protected. Within *Tile Group OBUs* or *FrameOBUs*, the following applies:

  - A subsample SHALL be created for each tile.
  - *BytesOfProtectedData* SHALL be a multiple of 16 bytes.
  - *BytesOfProtectedData* SHALL end on the last byte of the decode_tile structure (includ-ing any trailing bits).
  - *BytesOfProtectedData* SHALL span all complete 16-byte blocks of the decode_tile struc-ture (including any trailing bits).

```
NOTE: As a result of the above, partial blocks are not used and it is possible that BytesOfPro-tectedData does not start at the first byte of the decode_tile structure, but some number of bytes following that.
```

  - All other parts of *Tile Group OBUs* and *Frame OBUs* SHALL be unprotected.

## Subsample Encryption Illustration

<figure class="text-center">
      <img src="images/The multicast key derivation scheme.svg" alt="Subsample-based AV1 encryption">
      <figcaption>Subsample-based AV1 encryption</figcaption>
</figure>




### PackageVersionReq & Ans

The **_PackageVersionReq** command has no payload.
The end-device answers with a **_PackageVersionAns_** command with the following payload.

<table>
    <caption>PackageVersionAns</caption>
    <thead>
        <tr>
            <th>Field</th>
            <th>Size(bytes)</th>
        </tr>
        <tr>
            <td>PackageIdentifier</td>
            <td>1</td>
        </tr>
	<tr>
            <td>PackageVersion</td>
            <td>1</td>
        </tr>
    </tbody>
</table>

*PackageIdentifier uniquely* identifies the package. For the “multicast control package” this identifier is 2.

*PackageVersion* corresponds to the version of the package specification implemented by the end-device. 

### McGroupStatusReq & Ans

The McGroupStatusReq command has a single byte payload.

<table>
    <caption>McGroupStatusReq </caption>
    <thead>
        <tr>
            <th>Field</th>
            <th>Size(bytes)</th>
        </tr>
        <tr>
            <td>CmdMask</td>
            <td>1</td>
        </tr>
    </tbody>
</table>

Where:

<table>
    <caption>McGroupStatusReq CmdMask field</caption>
    <thead>
        <tr>
            <th>CmdMask Fields</th>
            <th>Size(bits)</th>
        </tr>
        <tr>
            <td>RFU</td>
            <td>4 bits</td>
        </tr>
	 <tr>
            <td>ReqGroupMask</td>
            <td>4 bits</td>
        </tr>
    </tbody>
</table>

The *ReqGroupMask* bit mask defines the multicast groups whose status should be reported by the end-device.  ReqGroupMask[n] = 1 means that the nth multicast group status SHOULD be included in the answer. ReqGroupMask[n] = 0 means that this group SHALL NOT be included in the answer.

The end-device responds to the McGroupStatusReq command with a McGroupStatusAns with the following payload:

<table>
    <caption> McGroupStatusAns</caption>
    <thead>
        <tr>
            <th>Field</th>
            <th>Size(bytes)</th>
        </tr>
        <tr>
            <td>status</td>
            <td>1</td>
        </tr>
	 <tr>
            <td>Optional list of [McGroupID+McAddr]</td>
            <td>5xNbItems</td>
        </tr>
    </tbody>
</table>

The status field encodes the following information:

<table>
    <caption>McGroupStatusAns Status field</caption>
    <thead>
        <tr>
            <th>Status Fields</th>
            <th>Size(bits)</th>
        </tr>
        <tr>
            <td>RFU</td>
            <td>1 bit</td>
        </tr>
	 <tr>
            <td>NbTotalGroups</td>
            <td>3bits</td>
        </tr>
	 <tr>
            <td>AnsGroupMask</td>
            <td>4bits</td>
        </tr>
    </tbody>
</table>

*AnsGroupMask* is a bit mask describing which groups are listed in the report. If the end-device cannot report the status of the multicast groups specified by the *ReqGroupMask* field of the request, the end-device SHALL discard the nth last groups (starting with the highest GroupID) until the answer fits. In that case, the *AnsGroupMask* mask is different from the *ReqGroupMask*. In that case the server can get the status of the groups not listed by issuing a new McGroupStatusReq command with another *RegGroupMask* field.  If all groups requested can be listed, *AnsGroupMask8* == *ReqGroupMask*.

*NbTotalGroups* is the number of multicast groups currently defined in the end-device. The valid range is [0:4].

Each record consists of 5 bytes [McGroupID + McAddr].
McGroupID and McAddr are provided to the end-device by McGroupSetupReq.

### McGroupSetupReq & Ans

This command is used to create or modify the parameters of a multicast group.
The payload of the message is:

<table>
    <caption>McGroupSetupReq</caption>
    <thead>
        <tr>
            <th>Field</th>
            <th>Size(bytes)</th>
        </tr>
        <tr>
            <td>McGroupIDHeader</td>
            <td>1</td>
        </tr>
	 <tr>
            <td>McAddr</td>
            <td>4</td>
        </tr>
	 <tr>
            <td>McKey_encrypted</td>
            <td>16</td>
        </tr>
	  <tr>
            <td>minMcFCount</td>
            <td>4</td>
        </tr>
	 <tr>
            <td>maxMcFCount</td>
            <td>4</td>
        </tr>
    </tbody>
</table>

Where:

<table>
    <caption>McGroupSetupReq McGroupIDHeader field</caption>
    <thead>
        <tr>
            <th>McGroupIDHeader Fields</th>
            <th>Size(bits)</th>
        </tr>
        <tr>
            <td>RFU</td>
            <td>6 bits</td>
        </tr>
	 <tr>
            <td>McGroupID</td>
            <td>2 bits</td>
        </tr>
    </tbody>
</table>

*McGroupID* is the multicast group ID of the multicast context. An end-device MAY support being part of several multicast group simultaneously. Therefore, all multicast related command MUST always contain an identifier (the McGroupID) of the multicast group being affected. 

```
Note: The McAddr could be used as a multicast group identifier but this would add a systematic 4
bytes overhead, so a more compact McGroupID is used. Additionally, if MultiCast keys are kept in a
Hardware Secure Element that can only keep a few keys, the MCU needs to indicate which key memory
slot should be used. Therefore, the Multicast group ID concept is required.
```
An end-device implementing this package SHALL support at least one multicast group. An end-device MAY support up to a maximum of 4 simultaneous multicast contexts.

*McKey_encrypted* is the encrypted multicast group key from which McAppSKey and McNetSKey will be derived. The McKey_encrypted key can be decrypted using the following operation to give the multicast group’s McKey.
	McKey = aes128_encrypt(McKEKey, McKey_encrypted)

The McKEKey is a __**lifetime end-device specific**__ key used to encrypt Multicast key transported over the air (it is a Key Encryption Key), and may be either:

* Derived from a new root key (GenAppKey) provisioned in the end-device at any time before the deployment of the end-device in the field. LoRaWAN 1.0.x end-devices SHALL use this scheme. 
  * McRootKey = aes128_encrypt(GenAppKey, 0x00 | pad16)
  * McKEKey = aes128_encrypt(McRootKey, 0x00 | pad16)

* Derived from the AppKey.  LoRaWAN 1.1+ end-devices SHALL use this scheme.
  * McRootKey = aes128_encrypt(AppKey, 0x20 | pad16)
  * McKEKey = aes128_encrypt(McRootKey, 0x00 | pad16)
  
The McAppSKey and the McNetSKey are then derived from the group’s McKey as follow:

McAppSKey = aes128_encrypt(McKey, 0x01 | McAddr | pad16)
McNetSKey = aes128_encrypt(McKey, 0x02 | McAddr | pad16)

The multicast key derivation scheme is summarized in the following diagram.

<figure class="text-center">
      <img src="images/The multicast key derivation scheme.svg" alt="The multicast key derivation scheme">
      <figcaption>The multicast key derivation scheme</figcaption>
</figure>

```
Note: using a Key Encryption Key to transport the multicast group McKey allows for a completely
secure multicast scheme when using a hardware secure element, when the secure element does not export
the McKey, McAppSKey, and McNwkSKey to the outside. It does not increase the security if a full
software implementation is used in the end-device. However, for compatibility reason it is
recommended to systematically use this scheme.
```

The *minMcFCount* field is the next frame counter value of the multicast downlink to be sent by the server for this group. This information is required in case an end-device is added to a group that already exists. The end-device MUST reject any downlink multicast frame using this group multicast address if the frame counter is < minMcFCount. 

*McAddr* is the multicast group network address. McAddr is negotiated off-band by the application server with the network server.

*maxMcFCount* specifies the life time of this multicast group expressed as a maximum number of frames. The end-device will only accept a multicast downlink frame if the 32bits frame counter value minMcFCount ≤ McFCount < maxMcFCount.

The end-device acknowledges the reception of this message by sending an **_McGroupSetupAns_** message with the following payload:

<table>
    <caption>McGroupSetupAns</caption>
    <thead>
        <tr>
            <th>Field</th>
            <th>Size (bytes)</th>
        </tr>
	<tr>
            <td>McGroupIDHeader</td>
            <td>1</td>
        </tr>
    </tbody>
</table>

Where:

<table>
    <caption>McGroupSetupAns McGroupIDHeader field</caption>
    <thead>
        <tr>
            <th>McGroupIDHeader Fields</th>
            <th>Size (bytes)</th>
        </tr>
        <tr>
            <td>RFU</td>
            <td>5</td>
        </tr>
	 <tr>
            <td>IDerror</td>
            <td>1</td>
        </tr>
	 <tr>
            <td>McGroupID</td>
            <td>2</td>
        </tr>
    </tbody>
</table>

When set the *IDerror* bit indicates that the end-device does not support the multicast context indexed by the McGroupID requested by the server. For example, an end-device MAY only support a single multicast group (McGroupID=0). If the server tries to create a second multicast group with McGroupID = 1, the end-device SHALL respond with IDerror=1.

### McGroupDeleteReq & Ans

This message is used to delete a multicast group from an end-device. The command payload is:

<table>
    <caption>McGroupDeleteReq</caption>
    <thead>
        <tr>
            <th>Field</th>
            <th>Size (bytes)</th>
        </tr>
        <tr>
            <td>McGroupIDHeader</td>
            <td>1</td>
        </tr>
    </tbody>
</table>

Where:

<table>
    <caption>McGroupDeleteReq McGroupIDHeader field</caption>
    <thead>
        <tr>
            <th>McGroupIDHeader Fields</th>
            <th>Size (bits)</th>
        </tr>
        <tr>
            <td>RFU</td>
            <td>6bits</td>
        </tr>
	<tr>
            <td>McGroupID</td>
            <td>2bits</td>
        </tr>
    </tbody>
</table>

The end-device answers with **McGroupDeleteAns** with payload:

<table>
    <caption>McGroupDeleteAns</caption>
    <thead>
        <tr>
            <th>Field</th>
            <th>Size (bytes)</th>
        </tr>
        <tr>
            <td>McGroupIDHeader</td>
            <td>1</td>
        </tr>
    </tbody>
</table>

Where:

<table>
    <caption>McGroupDeleteAns McGroupIDHeader field</caption>
    <thead>
        <tr>
            <th>McGroupIDHeader Fields</th>
            <th>Size (bits)</th>
        </tr>
        <tr>
            <td>RFU</td>
            <td>5bits</td>
        </tr>
	<tr>
            <td>MCGroupUndefined</td>
            <td>1bit</td>
        </tr>
	 <tr>
            <td>McGroupID</td>
            <td>2bits</td>
        </tr>
    </tbody>
</table>

*MCGroupUndefined* is set 1 if the McGroupID specified by the command is not defined in the end-device (was not created before calling the delete command).

### McClassCSessionReq & Ans

This message is only used to setup a temporary classC multicast session associated with a multicast context. 

The payload of the message is:

<table>
    <caption>McClassCSessionReq</caption>
    <thead>
        <tr>
            <th>Field</th>
            <th>Size (bytes)</th>
        </tr>
        <tr>
            <td>McGroupIDHeader</td>
            <td>1</td>
        </tr>
	<tr>
            <td>Session Time</td>
            <td>4</td>
        </tr>
	 <tr>
            <td>SessionTimeOut</td>
            <td>1</td>
        </tr>
	 <tr>
            <td>DLFrequ</td>
            <td>3</td>
        </tr>
	 <tr>
            <td>DR</td>
            <td>1</td>
        </tr>
    </tbody>
</table>

Where:

<table>
    <caption>McClassCSessionReq McGroupIDHeader field</caption>
    <thead>
        <tr>
            <th>McGroupIDHeader Fields</th>
            <th>Size (bits)</th>
        </tr>
        <tr>
            <td>RFU</td>
            <td>6bits</td>
        </tr>
	<tr>
            <td>McGroupID</td>
            <td>2bits</td>
        </tr>
    </tbody>
</table>

And where:

<table>
    <caption>McClassCSessionReq SessionTimeOut field</caption>
    <thead>
        <tr>
            <th>SessionTimeOut Fields</th>
            <th>Size (bits)</th>
        </tr>
        <tr>
            <td>RFU</td>
            <td>4bits</td>
        </tr>
	<tr>
            <td>TimeOut</td>
            <td>4bits</td>
        </tr>
    </tbody>
</table>

*McGroupID* is the identifier of the multicast group being used.

*SessionTime* is the start of the Class C window, and is expressed as the time in seconds since 00:00:00, Sunday 6th of January 1980 (start of the GPS epoch) modulo 2^32. Note that this is the same format as the Time field in the beacon frame.

*TimeOut* encodes the maximum length in seconds of the multicast session (max time the end-device stays in classC before reverting to class A to save battery)
The maximum duration in second is 2TimeOut (Example: TimeOut=8 means 256 seconds)
This is a maximum duration because the end-device’s application might decide to revert to class A before the end of the session, this decision is application specific. 

```
For example, the multicast session might be used to broadcast a firmware upgrade file. In that case
the end-device might end the multicast session has soon as the full file is received without waiting
for TimeOut.
```

*DlFrequ:* Encodes the frequency used for the multicast. This field is a 24 bits unsigned integer. The actual channel frequency in Hz is 100 x DlFrequ whereby values representing frequencies below 100 MHz are reserved for future use. This allows setting the frequency of a channel anywhere between 100 MHz to 1.67 GHz in 100 Hz steps.
This field has the same meaning and coding as LoRaWAN *NewChannelReq* MAC command ‘Freq’ field.

*DR:* index of the data rate used for the multicast. Uses the same look-up table than the one used by the LinkAdrReq MAC command of the LoRaWAN protocol.

The end-device acknowledges the reception of this message by sending a **McClassCSessionAns** message on the same port with the following payload:

<table>
    <caption>McClassCSessionAns</caption>
    <thead>
        <tr>
            <th>Field</th>
            <th>Size (bytes)</th>
        </tr>
        <tr>
            <td>Status&McGroupID</td>
            <td>1</td>
        </tr>
	<tr>
            <td>(cond)TimeToStart</td>
            <td>3</td>
        </tr>
    </tbody>
</table>

Where:

<table>
    <caption>McClassCSessionAns Status&McGroupID field</caption>
    <thead>
        <tr>
            <th>Status&McGroupID Fields</th>
            <th>Size (bits)</th>
        </tr>
        <tr>
            <td>RFU</td>
            <td>3bits</td>
        </tr>
	<tr>
            <td>McGroupUndefined</td>
            <td>1bit</td>
        </tr>	
	<tr>
            <td>FreqError</td>
            <td>1bit</td>
        </tr>	
	 <tr>
            <td>DRError</td>
            <td>1bit</td>
        </tr>	
	 <tr>
            <td>McGroupID</td>
            <td>1bit</td>
        </tr>
    </tbody>
</table>

*FreqError* bit is set to 1 if the DLFrequ frequency set by the network is not usable for the end-device.

*DRError* bit is set to 1 if the classC downlink Data Rate set by the network is not defined.

*McGroupUndefined* is set 1 if the McGroupID specified by the command is not defined in the end-device (was not created before calling this command).

If no errors are present, the *TimeToStart* field encodes the number of seconds from the **_McClassCSessionAns_** uplink to the beginning of the multicast session. This allows the server to check that the end-device clock is well synchronized and that the end-device will effectively switch to classC exactly at the right moment (with second accuracy). This is possible because all uplinks are accurately time stamped by the network gateways (at least with an accuracy better than the second).

### McClassBSessionReq & Ans

This message is only used to setup a temporary ClassB multicast session associated with a multicast context. 
The payload of the message is:

<table>
    <caption>McClassBSessionReq</caption>
    <thead>
        <tr>
            <th>Field</th>
            <th>Size (bytes)</th>
        </tr>
	<tr>
            <td>McGroupIDHeader</td>
            <td>1</td>
        </tr>	
	<tr>
            <td>Session Time</td>
            <td>4</td>
        </tr>	
	 <tr>
            <td>TimeOutPeriodicity</td>
            <td>1</td>
        </tr>	
	 <tr>
            <td>DLFrequ</td>
            <td>3</td>
        </tr>
	<tr>
            <td>DR</td>
            <td>1</td>
        </tr>
    </tbody>
</table>

Where:

<table>
    <caption>McClassBSessionReq McGroupIDHeader field</caption>
    <thead>
        <tr>
            <th>McGroupIDHeader Fields</th>
            <th>Size (bits)</th>
        </tr>
        <tr>
            <td>RFU</td>
            <td>6bits</td>
        </tr>
	<tr>
            <td>McGroupID</td>
            <td>2bits</td>
        </tr>	
    </tbody>
</table>

And where:

<table>
    <caption>McClassBSessionReq TimeOutPeriodicity field</caption>
    <thead>
        <tr>
            <th>TimeOutPeriodicity Fields</th>
            <th>Size (bits)</th>
        </tr>
        <tr>
            <td>RFU</td>
            <td>1bits</td>
        </tr>
	<tr>
            <td>Periodicity</td>
            <td>3bits</td>
        </tr>
	<tr>
            <td>TimeOut</td>
            <td>4bits</td>
        </tr>
    </tbody>
</table>

*McGroupID* is the identifier of the multicast group being used.

*SessionTime* is the start of the Class B window, and is expressed as the time in seconds since 00:00:00, Sunday 6th of January 1980 (start of the GPS epoch) modulo 2^32. Note that this is the same format as the Time field in the beacon frame. SessionTime MUST be an integer multiple of 128.

*TimeOut* encodes the maximum length in BeaconPeriods (128seconds) of the multicast fragmentation session (max time the end-device stays in classB before eventually reverting to class A to save battery)
The maximum duration in second is 128*2TimeOut (Example: TimeOut=8 corresponds roughly to 9.1hours).

```
Attention: For classB TimeOut is expressed in BeaconPeriod (128sec), whereas it is expressed in 
seconds for classC. This is because a classB multicast session is heavily duty-cycled and is likely
to last a lot longer than a classC session.
```
This is a maximum duration because the end-device’s application might decide to revert to class A before the end of the session, this decision is application specific. 

*Periodicity* encodes the classB ping slot periodicity for the multicast group. The encoding format is the same than for the Periodicity field of the **_PingSlotInfoReq_** classB MAC command defined in LoRaWAN.

*DlFrequ*: Encodes the frequency used for the multicast. This field is a 24 bits unsigned integer. The actual channel frequency in Hz is 100 x DlFrequ whereby values representing frequencies below 100 MHz are reserved for future use. This allows setting the frequency of a channel anywhere between 100 MHz to 1.67 GHz in 100 Hz steps. 
This field has the same meaning and coding as LoRaWAN *NewChannelReq* MAC command 'Freq' field.

In regions where the classB beacon is transmitted following a frequency hopping pattern, DlFrequ=0 signals the end-device to use the default classB default frequency hopping scheme. That scheme is defined in the classB section of the LoRaWAN specification. In that case, Class B downlinks use a channel which is a function of the Time field of the last beacon (see Beacon Frame content) and the multicast address McAddr.

```
Class B downlink channel=[McAddr+floor ((Beacon_Time )/(Beacon_period))] modulo NbChannel
```
* Whereby Beacon_Time is the 32-bit Time field of the current beacon period
* Beacon_period is the length of the beacon period (defined as 128sec in the specification)
* Floor designates rounding to the immediately lower integer value
* McAddr is the 32-bit network address of the multicast group
* NbChannel is the number of channel over which the beacon is frequency hopping
 
*DR*: index of the data rate used for the classB multicast. Uses the same look-up table than the one used by the LinkAdrReq MAC command of the LoRaWAN protocol.

The end-device acknowledges the reception of this message by sending a **_McClassBSessionAns** message on the same port with the following payload:

<table>
    <caption>McClassBSessionAns</caption>
    <thead>
        <tr>
            <th>Field</th>
            <th>Size (bytes)</th>
        </tr>
        <tr>
            <td>Status&McGroupID</td>
            <td>1</td>
        </tr>
	<tr>
            <td>(cond)TimeToStart</td>
            <td>3</td>
        </tr>
    </tbody>
</table>

Where:

<table>
    <caption>McClassBSessionAns Status&McGroupID field</caption>
    <thead>
        <tr>
            <th>Field</th>
            <th>Size (bits)</th>
        </tr>
        <tr>
            <td>RFU</td>
            <td>3bits</td>
        </tr>
	<tr>
            <td>McGroupUndefined</td>
            <td>1bit</td>
        </tr>
	<tr>
            <td>FreqError</td>
            <td>1bit</td>
        </tr>
	<tr>
            <td>DRError</td>
            <td>1bit</td>
        </tr>
	<tr>
            <td>McGroupID</td>
            <td>2bit</td>
        </tr>
    </tbody>
</table>

*FreqError* bit is set to 1 if the DLFrequ frequency set by the network is not usable for the end-device.

*DRError* bit is set to 1 if the classB downlink Data Rate set by the network is not defined.

*McGroupUndefined* is set 1 if the McGroupID specified by the command is not defined in the end-device (was not created before calling this command).

If no errors are present, the TimeToStart field encodes the number of seconds from the **_McClassBSessionAns_** uplink to the beginning of the multicast fragmentation session. This allows the server to check that the end-device clock is roughly synchronized and that it will effectively start acquiring the classB beacon at the right moment (before the beginning of the classB multicast session with some margin).

## Glossary


<table>
    <caption>Glossary</caption>
    <thead>
        <tr>
            <th>Abbriviation</th>
            <th>Meaning</th>
        </tr>
        <tr>
            <td>AS</td>
            <td>Application Server</td>
        </tr>
	<tr>
            <td>TBD</td>
            <td>To Be Done</td>
        </tr>
    </tbody>
</table>

## Bibliography 

### References

[LoRaWAN 1.0.2]: LoRaWANTM 1.0.2 Specification, LoRa Alliance, July 2016

[LoRaWAN 1.1]: LoRaWANTM 1.1 Specification, LoRa Alliance, October 11, 2017

## NOTICE OF USE AND DISCLOSURE

Copyright © LoRa Alliance, Inc. (2018). All Rights Reserved.

The information within this document is the property of the LoRa Alliance (“The Alliance”) and its use and disclosure are subject to LoRa Alliance Corporate Bylaws, Intellectual Property Rights (IPR) Policy and Membership Agreements.

Elements of LoRa Alliance specifications may be subject to third party intellectual property rights, including without limitation, patent, copyright or trademark rights (such a third party may or may not be a member of LoRa Alliance). The Alliance is not responsible and shall not be held responsible in any manner for identifying or failing to identify any or all such third party intellectual property rights.

This document and the information contained herein are provided on an “AS IS” basis and THE ALLIANCE DISCLAIMS ALL WARRANTIES EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO (A) ANY WARRANTY THAT THE USE OF THE INFORMATION HEREIN WILL NOT INFRINGE ANY RIGHTS OF THIRD PARTIES (INCLUDING WITHOUT LIMITATION ANY INTELLECTUAL PROPERTY RIGHTS INCLUDING PATENT, COPYRIGHT OR TRADEMARK RIGHTS) OR (B) ANY IMPLIED WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE, TITLE OR NONINFRINGEMENT.

IN NO EVENT WILL THE ALLIANCE BE LIABLE FOR ANY LOSS OF PROFITS, LOSS OF BUSINESS, LOSS OF USE OF DATA, INTERRUPTION OFBUSINESS, OR FOR ANY OTHER DIRECT, INDIRECT, SPECIAL OR EXEMPLARY, INCIDENTIAL, PUNITIVE OR CONSEQUENTIAL DAMAGES OF ANY KIND, IN CONTRACT OR IN TORT, IN CONNECTION WITH THIS DOCUMENT OR THE INFORMATION CONTAINED HEREIN, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH LOSS OR DAMAGE. 

The above notice and this paragraph must be included on all copies of this document that are made.

LoRa Alliance™
5177 Brandin Court
Fremont, CA 94538
United States

Note: All Company, brand and product names may be trademarks that are the sole property of their respective owners.
