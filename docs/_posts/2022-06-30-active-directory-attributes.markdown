---
layout: single
title:  "Active Directory - Schema Updates and Attributes"
date:   2022-06-30 17:50:00 +1000
toc: true
categories: active directory schema attributes powershell
---

Adding new attributes to Active Directory can be done in a couple of different ways.  One method can use the MMC or alternatively use powershell.  When performing schema updates you will need an OID root to add new attributes.

NOTE that changes to the schema are pemenant and cannot be removed once they have been added so be careful to get it right the first time.

# Obtaining an OID root

A script is available from Microsoft to generate a random OID root that can be used to add attributes.  The script is written in visual basic and without modification will dump a text file on the desktop with the randonly generated root OID.  The script can be found [here][ms-oid-generator].

# Using MMC

The MMC provides a GUI for performing schema updates which is useful for small changes.   A good guide for this process can be found [here][mmc-schema-updates].

# Using Powershell

An example powershell script that can be used to generate a number of attributes is shown below.  In this example the OID has been manually set using the OID generator.  `$attributeNames` defines a hashtable of all the attributes to be added.  The `oMSyntax` and `attributeSyntax` fields define the type of the attribute and these values can be determined from the [syntaxes][ms-schema-syntaxes] page from Microsoft.

The second half of the script adds the attribute to the "user" schema.

Not shown in this script but if creating a large amount of related attributes then should consider adding a [class][ms-schema-class].

{% highlight powershell %}
$schemaPath = (Get-ADRootDSE).schemaNamingContext
$userSchema = get-adobject -SearchBase $schemapath -Filter 'name -eq "user"'

$oidRoot = "1.2.840.113556.1.8000.2554.83.55525.3724.17190.46825.13888324.999999"

$attributeNames = @( 
		@{ 'lDAPDisplayName' = 'labTokenUser'; 'oMSyntax' = 1; 'attributeSyntax' = '2.5.5.8'; 'isSingleValued' = $true; searchflags = 1; }, 
		@{ 'lDAPDisplayName' = 'labTokenStartDate'; 'oMSyntax' = 24; 'attributeSyntax' = '2.5.5.11'; 'isSingleValued' = $true; searchflags = 1; },
		@{ 'lDAPDisplayName' = 'labTokenGenericAttribute'; 'oMSyntax' = 27; 'attributeSyntax' = '2.5.5.3'; 'isSingleValued' = $false; searchflags = 1; },
		
		@{ 'lDAPDisplayName' = 'labTokenEnabled1'; 'oMSyntax' = 1; 'attributeSyntax' = '2.5.5.8'; 'isSingleValued' = $true; searchflags = 1; },
		@{ 'lDAPDisplayName' = 'labTokenEnrolled1'; 'oMSyntax' = 1; 'attributeSyntax' = '2.5.5.8'; 'isSingleValued' = $true; searchflags = 1; },
		@{ 'lDAPDisplayName' = 'labTokenGenericAttribute1'; 'oMSyntax' = 27; 'attributeSyntax' = '2.5.5.3'; 'isSingleValued' = $false; searchflags = 1; },
		@{ 'lDAPDisplayName' = 'labTokenSerial1'; 'oMSyntax' = 27; 'attributeSyntax' = '2.5.5.3'; 'isSingleValued' = $true; searchflags = 1; },
		
		@{ 'lDAPDisplayName' = 'labTokenEnabled2'; 'oMSyntax' = 1; 'attributeSyntax' = '2.5.5.8'; 'isSingleValued' = $true; searchflags = 1; },
		@{ 'lDAPDisplayName' = 'labTokenEnrolled2'; 'oMSyntax' = 1; 'attributeSyntax' = '2.5.5.8'; 'isSingleValued' = $true; searchflags = 1; },
		@{ 'lDAPDisplayName' = 'labTokenGenericAttribute2'; 'oMSyntax' = 27; 'attributeSyntax' = '2.5.5.3'; 'isSingleValued' = $false; searchflags = 1; },
		@{ 'lDAPDisplayName' = 'labTokenSerial2'; 'oMSyntax' = 27; 'attributeSyntax' = '2.5.5.3'; 'isSingleValued' = $true; searchflags = 1; },
		
		@{ 'lDAPDisplayName' = 'labTokenEnabled3'; 'oMSyntax' = 1; 'attributeSyntax' = '2.5.5.8'; 'isSingleValued' = $true; searchflags = 1; },
		@{ 'lDAPDisplayName' = 'labTokenEnrolled3'; 'oMSyntax' = 1; 'attributeSyntax' = '2.5.5.8'; 'isSingleValued' = $true; searchflags = 1; },
		@{ 'lDAPDisplayName' = 'labTokenGenericAttribute3'; 'oMSyntax' = 27; 'attributeSyntax' = '2.5.5.3'; 'isSingleValued' = $false; searchflags = 1; },
		@{ 'lDAPDisplayName' = 'labTokenSerial3'; 'oMSyntax' = 27; 'attributeSyntax' = '2.5.5.3'; 'isSingleValued' = $true; searchflags = 1; }
		);
		
$attrIndex = 1
foreach ($attr in $attributeNames) {

	$attr['attributeId'] = "$oidRoot.$attrIndex"

	write-Output $attr
	
	New-ADObject -Name $attr['lDAPDisplayName'] -Type attributeSchema -Path $schemapath -OtherAttributes $attr
	$userSchema | Set-ADObject -Add @{mayContain = $attr['lDAPDisplayName']} 
	
	$attrIndex = $attrIndex + 1
}


{% endhighlight %}


[ms-oid-generator]: https://docs.microsoft.com/en-us/windows/win32/ad/obtaining-an-object-identifier-from-microsoft
[mmc-schema-updates]: https://social.technet.microsoft.com/wiki/contents/articles/20319.how-to-create-a-custom-attribute-in-active-directory.aspx
[ms-schema-syntaxes]: https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-2000-server/cc961740(v=technet.10)?redirectedfrom=MSDN
[ms-schema-class]: https://docs.microsoft.com/en-us/windows/win32/ad/defining-a-new-class