# Third-Party Open-Source Notices

StemRuntimeSDK 1.0.0 for Android incorporates the open-source
libraries listed below. Each remains governed by its own license. The
Apache License, Version 2.0, is reproduced verbatim at the end of this
file; the per-component entries in the table below carry each
component's own copyright notice, as required by that license.

## Components

| # | Library | Version | License | Copyright notice |
|---|---------|---------|---------|------------------|
| 1 | [kotlin-stdlib](https://github.com/JetBrains/kotlin) | 2.1.0 | Apache-2.0 | Copyright 2010-2024 JetBrains s.r.o. and Kotlin Programming Language contributors |
| 2 | [kotlinx.coroutines](https://github.com/Kotlin/kotlinx.coroutines) (`-core`, `-android`, `-play-services`) | 1.9.0 | Apache-2.0 | Copyright 2016-2024 JetBrains s.r.o. and contributors |
| 3 | [kotlinx.serialization](https://github.com/Kotlin/kotlinx.serialization) (`-json`) | 1.7.3 | Apache-2.0 | Copyright 2017-2024 JetBrains s.r.o. and Kotlin Programming Language contributors |
| 4 | [kotlinx-datetime](https://github.com/Kotlin/kotlinx-datetime) | 0.6.1 | Apache-2.0 | Copyright 2019-2024 JetBrains s.r.o. and contributors |
| 5 | [androidx.core:core-ktx](https://developer.android.com/jetpack/androidx/releases/core) | 1.15.0 | Apache-2.0 | Copyright (c) The Android Open Source Project |
| 6 | [androidx.lifecycle](https://developer.android.com/jetpack/androidx/releases/lifecycle) (`-runtime-ktx`, `-process`, `-viewmodel-compose`) | 2.8.7 | Apache-2.0 | Copyright (c) The Android Open Source Project |
| 7 | [androidx.activity:activity-compose](https://developer.android.com/jetpack/androidx/releases/activity) | 1.9.3 | Apache-2.0 | Copyright (c) The Android Open Source Project |
| 8 | [androidx.navigation:navigation-compose](https://developer.android.com/jetpack/androidx/releases/navigation) | 2.8.5 | Apache-2.0 | Copyright (c) The Android Open Source Project |
| 9 | [androidx.compose](https://developer.android.com/jetpack/androidx/releases/compose) (BoM-pinned: `ui`, `foundation`, `material3`, `material-icons-extended`, `ui-tooling-preview`) | BoM 2024.12.01 | Apache-2.0 | Copyright (c) The Android Open Source Project |
| 10 | [androidx.media3](https://developer.android.com/media/media3) (`-exoplayer`, `-exoplayer-hls`, `-ui`) | 1.5.1 | Apache-2.0 | Copyright (c) The Android Open Source Project |
| 11 | [androidx.security:security-crypto](https://developer.android.com/jetpack/androidx/releases/security) | 1.1.0-alpha06 | Apache-2.0 | Copyright (c) The Android Open Source Project |
| 12 | [Coil](https://github.com/coil-kt/coil) (`coil-compose`, `coil-network-okhttp`) | 3.0.4 | Apache-2.0 | Copyright 2024 Coil Contributors |
| 13 | [Tink](https://github.com/tink-crypto/tink-java) (`tink-android`) | 1.16.0 | Apache-2.0 | Copyright (c) Google Inc. |
| 14 | [play-services-location](https://developers.google.com/android/guides/setup) | 21.3.0 | [Android SDK License Agreement](https://developer.android.com/studio/terms) | Copyright (c) Google LLC |

No modifications have been made by Licensor to any of these
components. Source code at the pinned versions is available from the
URLs above.

If you believe an attribution is incorrect or missing, please email
[vkrychun@stemjson.com](mailto:vkrychun@stemjson.com) or open an
issue at
[github.com/vkrychun/stem-runtime-kotlin](https://github.com/vkrychun/stem-runtime-kotlin).

---

## Apache License, Version 2.0

Applies to components #1–13. Per Section 4(a), the full text of the
Apache License, Version 2.0 is reproduced below. Per Section 4(c), the
copyright notice for each component is reproduced in the table above.
Per Section 4(d), any attribution text supplied in an upstream
`NOTICE` file at the pinned version is reproduced in the "Apache-2.0
NOTICE content" section below.

```
                                 Apache License
                           Version 2.0, January 2004
                        http://www.apache.org/licenses/

   TERMS AND CONDITIONS FOR USE, REPRODUCTION, AND DISTRIBUTION

   1. Definitions.

      "License" shall mean the terms and conditions for use, reproduction,
      and distribution as defined by Sections 1 through 9 of this document.

      "Licensor" shall mean the copyright owner or entity authorized by
      the copyright owner that is granting the License.

      "Legal Entity" shall mean the union of the acting entity and all
      other entities that control, are controlled by, or are under common
      control with that entity. For the purposes of this definition,
      "control" means (i) the power, direct or indirect, to cause the
      direction or management of such entity, whether by contract or
      otherwise, or (ii) ownership of fifty percent (50%) or more of the
      outstanding shares, or (iii) beneficial ownership of such entity.

      "You" (or "Your") shall mean an individual or Legal Entity
      exercising permissions granted by this License.

      "Source" form shall mean the preferred form for making modifications,
      including but not limited to software source code, documentation
      source, and configuration files.

      "Object" form shall mean any form resulting from mechanical
      transformation or translation of a Source form, including but
      not limited to compiled object code, generated documentation,
      and conversions to other media types.

      "Work" shall mean the work of authorship, whether in Source or
      Object form, made available under the License, as indicated by a
      copyright notice that is included in or attached to the work
      (an example is provided in the Appendix below).

      "Derivative Works" shall mean any work, whether in Source or Object
      form, that is based on (or derived from) the Work and for which the
      editorial revisions, annotations, elaborations, or other modifications
      represent, as a whole, an original work of authorship. For the purposes
      of this License, Derivative Works shall not include works that remain
      separable from, or merely link (or bind by name) to the interfaces of,
      the Work and Derivative Works thereof.

      "Contribution" shall mean any work of authorship, including
      the original version of the Work and any modifications or additions
      to that Work or Derivative Works thereof, that is intentionally
      submitted to Licensor for inclusion in the Work by the copyright owner
      or by an individual or Legal Entity authorized to submit on behalf of
      the copyright owner. For the purposes of this definition, "submitted"
      means any form of electronic, verbal, or written communication sent
      to the Licensor or its representatives, including but not limited to
      communication on electronic mailing lists, source code control systems,
      and issue tracking systems that are managed by, or on behalf of, the
      Licensor for the purpose of discussing and improving the Work, but
      excluding communication that is conspicuously marked or otherwise
      designated in writing by the copyright owner as "Not a Contribution."

      "Contributor" shall mean Licensor and any individual or Legal Entity
      on behalf of whom a Contribution has been received by Licensor and
      subsequently incorporated within the Work.

   2. Grant of Copyright License. Subject to the terms and conditions of
      this License, each Contributor hereby grants to You a perpetual,
      worldwide, non-exclusive, no-charge, royalty-free, irrevocable
      copyright license to reproduce, prepare Derivative Works of,
      publicly display, publicly perform, sublicense, and distribute the
      Work and such Derivative Works in Source or Object form.

   3. Grant of Patent License. Subject to the terms and conditions of
      this License, each Contributor hereby grants to You a perpetual,
      worldwide, non-exclusive, no-charge, royalty-free, irrevocable
      (except as stated in this section) patent license to make, have made,
      use, offer to sell, sell, import, and otherwise transfer the Work,
      where such license applies only to those patent claims licensable
      by such Contributor that are necessarily infringed by their
      Contribution(s) alone or by combination of their Contribution(s)
      with the Work to which such Contribution(s) was submitted. If You
      institute patent litigation against any entity (including a
      cross-claim or counterclaim in a lawsuit) alleging that the Work
      or a Contribution incorporated within the Work constitutes direct
      or contributory patent infringement, then any patent licenses
      granted to You under this License for that Work shall terminate
      as of the date such litigation is filed.

   4. Redistribution. You may reproduce and distribute copies of the
      Work or Derivative Works thereof in any medium, with or without
      modifications, and in Source or Object form, provided that You
      meet the following conditions:

      (a) You must give any other recipients of the Work or
          Derivative Works a copy of this License; and

      (b) You must cause any modified files to carry prominent notices
          stating that You changed the files; and

      (c) You must retain, in the Source form of any Derivative Works
          that You distribute, all copyright, patent, trademark, and
          attribution notices from the Source form of the Work,
          excluding those notices that do not pertain to any part of
          the Derivative Works; and

      (d) If the Work includes a "NOTICE" text file as part of its
          distribution, then any Derivative Works that You distribute must
          include a readable copy of the attribution notices contained
          within such NOTICE file, excluding those notices that do not
          pertain to any part of the Derivative Works, in at least one
          of the following places: within a NOTICE text file distributed
          as part of the Derivative Works; within the Source form or
          documentation, if provided along with the Derivative Works; or,
          within a display generated by the Derivative Works, if and
          wherever such third-party notices normally appear. The contents
          of the NOTICE file are for informational purposes only and
          do not modify the License. You may add Your own attribution
          notices within Derivative Works that You distribute, alongside
          or as an addendum to the NOTICE text from the Work, provided
          that such additional attribution notices cannot be construed
          as modifying the License.

      You may add Your own copyright statement to Your modifications and
      may provide additional or different license terms and conditions
      for use, reproduction, or distribution of Your modifications, or
      for any such Derivative Works as a whole, provided Your use,
      reproduction, and distribution of the Work otherwise complies with
      the conditions stated in this License.

   5. Submission of Contributions. Unless You explicitly state otherwise,
      any Contribution intentionally submitted for inclusion in the Work
      by You to the Licensor shall be under the terms and conditions of
      this License, without any additional terms or conditions.
      Notwithstanding the above, nothing herein shall supersede or modify
      the terms of any separate license agreement you may have executed
      with Licensor regarding such Contributions.

   6. Trademarks. This License does not grant permission to use the trade
      names, trademarks, service marks, or product names of the Licensor,
      except as required for describing the origin of the Work and
      reproducing the content of the NOTICE file.

   7. Disclaimer of Warranty. Unless required by applicable law or
      agreed to in writing, Licensor provides the Work (and each
      Contributor provides its Contributions) on an "AS IS" BASIS,
      WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or
      implied, including, without limitation, any warranties or conditions
      of TITLE, NON-INFRINGEMENT, MERCHANTABILITY, or FITNESS FOR A
      PARTICULAR PURPOSE. You are solely responsible for determining the
      appropriateness of using or redistributing the Work and assume any
      risks associated with Your exercise of permissions under this License.

   8. Limitation of Liability. In no event and under no legal theory,
      whether in tort (including negligence), contract, or otherwise,
      unless required by applicable law (such as deliberate and grossly
      negligent acts) or agreed to in writing, shall any Contributor be
      liable to You for damages, including any direct, indirect, special,
      incidental, or consequential damages of any character arising as a
      result of this License or out of the use or inability to use the
      Work (including but not limited to damages for loss of goodwill,
      work stoppage, computer failure or malfunction, or any and all
      other commercial damages or losses), even if such Contributor
      has been advised of the possibility of such damages.

   9. Accepting Warranty or Additional Liability. While redistributing
      the Work or Derivative Works thereof, You may choose to offer,
      and charge a fee for, acceptance of support, warranty, indemnity,
      or other liability obligations and/or rights consistent with this
      License. However, in accepting such obligations, You may act only
      on Your own behalf and on Your sole responsibility, not on behalf
      of any other Contributor, and only if You agree to indemnify,
      defend, and hold each Contributor harmless for any liability
      incurred by, or claims asserted against, such Contributor by reason
      of your accepting any such warranty or additional liability.

   END OF TERMS AND CONDITIONS
```

### Apache-2.0 NOTICE content

The AndroidX components (Jetpack libraries) at their pinned versions
ship a `NOTICE` file with the following content, reproduced here to
satisfy Section 4(d) of the Apache License, Version 2.0:

```
Copyright (C) The Android Open Source Project

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

      https://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```

---

## Android SDK License Agreement

Applies to component #14 (`play-services-location`). The
`play-services-location` artefact is distributed by Google under the
[Android SDK License Agreement](https://developer.android.com/studio/terms),
not Apache-2.0. Use of the artefact constitutes acceptance of those
terms; this Agreement does not grant any rights in that component
beyond those granted by Google.
