---
Author: cptnyan
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5749
Published Date: 2015-02-20t21
Archived Date: 2015-03-23t19
---

# new-errorrecord - 

## Description

creates an error record for throwing better/customized errors in scripts/modules. based on the new-errorrecord from http

## Comments



## Usage



## TODO



## script

`new-errorrecord`

## Code

`#
 #
 <#
 .SYNOPSIS
     Creates a custom error record.
 .DESCRIPTION
     This cmdlet generates a custom error record. Custom error 
     records can be used to provide additional or tailored 
     information to the user when encountering an exception.
 .PARAMETER Message
     The text error message to display to the user. This should
     be desciptive of the error encountered.
 .PARAMETER Exception
     The name of an exception. Valid exception names are:
         AccessViolationException
         AggregateException
         AppDomainUnloadedException
         ApplicationException
         ArgumentException
         ArgumentNullException
         ArgumentOutOfRangeException
         ArithmeticException
         ArrayTypeMismatchException
         BadImageFormatException
         CannotUnloadAppDomainException
         ContextMarshalException
         DataMisalignedException
         DivideByZeroException
         DllNotFoundException
         DuplicateWaitObjectException
         EntryPointNotFoundException
         Exception
         ExecutionEngineException
         FieldAccessException
         FormatException
         IndexOutOfRangeException
         InsufficientExecutionStackException
         InsufficientMemoryException
         InvalidCastException
         InvalidOperationException
         InvalidProgramException
         InvalidTimeZoneException
         MemberAccessException
         MethodAccessException
         MissingFieldException
         MissingMemberException
         MissingMethodException
         MulticastNotSupportedException
         NotFiniteNumberException
         NotImplementedException
         NotSupportedException
         NullReferenceException
         OperationCanceledException
         OutOfMemoryException
         OverflowException
         PlatformNotSupportedException
         RankException
         StackOverflowException
         SystemException
         TimeoutException
         TimeZoneNotFoundException
         TypeAccessException
         TypeLoadException
         TypeUnloadedException
         UnauthorizedAccessException
         UriFormatException
         UriTemplateMatchException
 .PARAMETER ErrorID
     A string identifying the error. This is freeform, but should not be 
     the same as an alredy defined error ID. In native PowerShell errors,
     this parameter sets the 'FullyQualifiedErrorID' field.
 .PARAMETER ErrorCategory
     An error category from [System.Management.Automation.ErrorCategory]
     that categorizes the error being thrown.
 .PARAMETER TargetObject
     The object that was being processed when the error was encountered.
 .PARAMETER InnerException
     The exception that caused the exception triggering the creation of
     this error record.
 .EXAMPLE
     New-ErrorRecord -Exception InvalidOperationException -Message 'The value provided in CollectionName is invalid.' -ErrorCategory InvalidArgument -ErrorID 'InvalidCollectionIdentifier'
 
     The value provided in CollectionName is invalid.
         + CategoryInfo          : InvalidArgument: (:) [], InvalidOperationException
         + FullyQualifiedErrorId : InvalidCollectionIdentifier
 
     The command above creates an InvalidOperationException error record and outputs a meaningful 
     message to the screen.
 .EXAMPLE
     New-ErrorRecord 'This is the error message' UnauthorizedAccessException 'ErrorID' PermissionDenied
     
     This is the error message
         + CategoryInfo          : PermissionDenied: (:) [], UnauthorizedAccessException
         + FullyQualifiedErrorId : ErrorID
     
     The above command generates an UnauthorizedAccessException error record using positional arguments.
 .EXAMPLE
     try
     {
         17 / 0
     }
     catch
     {
         New-ErrorRecord -Message 'Attempted to divide by zero.' -Exception DivideByZeroException -ErrorID 'DivideByZero' -ErrorCategory InvalidArgument -InnerException $PSItem.InnerException
     }
     Attempted to divide by zero.
         + CategoryInfo          : InvalidArgument: (:) [], DivideByZeroException
         + FullyQualifiedErrorId : DivideByZero
 
     This contrived example makes use of the InnerException parameter.
 .OUTPUTS
     [System.Management.Automation.ErrorRecord]
 #>
 function New-ErrorRecord
 {
     [CmdletBinding()]
 
     param
     (
         [Parameter( Mandatory = $true,
                     ValueFromPipeline = $true,
                     ValueFromPipelineByPropertyName = $true,
                     Position = 0 )]
         [String] $Message,
         [Parameter( Mandatory = $true,
                     ValueFromPipeline = $true,
                     ValueFromPipelineByPropertyName = $true,
                     Position = 1 )]
         [ValidateSet( 'AccessViolationException','AggregateException','AppDomainUnloadedException','ApplicationException',
                       'ArgumentException','ArgumentNullException','ArgumentOutOfRangeException','ArithmeticException',
                       'ArrayTypeMismatchException','BadImageFormatException','CannotUnloadAppDomainException',
                       'ContextMarshalException','DataMisalignedException','DivideByZeroException','DllNotFoundException',
                       'DuplicateWaitObjectException','EntryPointNotFoundException','Exception','ExecutionEngineException',
                       'FieldAccessException','FormatException','IndexOutOfRangeException','InsufficientExecutionStackException',
                       'InsufficientMemoryException','InvalidCastException','InvalidOperationException','InvalidProgramException',
                       'InvalidTimeZoneException','MemberAccessException','MethodAccessException','MissingFieldException',
                       'MissingMemberException','MissingMethodException','MulticastNotSupportedException','NotFiniteNumberException',
                       'NotImplementedException','NotSupportedException','NullReferenceException','OperationCanceledException',
                       'OutOfMemoryException','OverflowException','PlatformNotSupportedException','RankException','StackOverflowException',
                       'SystemException','TimeoutException','TimeZoneNotFoundException','TypeAccessException','TypeLoadException',
                       'TypeUnloadedException','UnauthorizedAccessException','UriFormatException','UriTemplateMatchException')]
         [String] $Exception,        
         [Parameter( Mandatory = $true,
                     ValueFromPipeline = $true,
                     ValueFromPipelineByPropertyName = $true,
                     Position = 2 )]
         [String] $ErrorID,
         [Parameter( Mandatory = $true,
                     ValueFromPipeline = $true,
                     ValueFromPipelineByPropertyName = $true,
                     Position = 3 )]
         [System.Management.Automation.ErrorCategory] $ErrorCategory,
         
         [Parameter( ValueFromPipeline = $true,
                     ValueFromPipelineByPropertyName = $true )]
         [System.Object] $TargetObject,
         [Parameter( ValueFromPipeline = $true,
                     ValueFromPipelineByPropertyName = $true )]
         [System.Exception] $InnerException
     )
     process 
     {
         $_Exception = New-Object $Exception -ArgumentList $Message,$InnerException
         $ErrorRecord = New-Object System.Management.Automation.ErrorRecord -ArgumentList $_Exception, $ErrorID, $ErrorCategory, $TargetObject
         Write-Output $ErrorRecord
     }
 }
`

