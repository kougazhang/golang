```
@startuml

abstract StandardFusion {
  CreateTasks() []ITask 
  Run([]ITask)
}

interface IFile {
  Upload(src io.Reader, dst string) error
  Download(src string, dst io.WriteCloser) error
}

interface ITransform {
  	Submit(param interface{}) (result interface{}, err error)
	GetStatus(interface{}) (Status, error)
}

interface ITask {
  Run(*Context) error
}


class JobKkmh
class JobGifshowLocal
class JobGifshowV3

class TransformFlink
class TransformLocal

class FileHttp
class FileHdfs
class FileUpyunStore


StandardFusion ..> IFile
StandardFusion ..> ITransform
StandardFusion ..> ITask

JobKkmh -down-|> StandardFusion
JobGifshowLocal -down-|> StandardFusion
JobGifshowV3 -down-|> StandardFusion

TransformFlink -up-|> ITransform
TransformLocal -up-|> ITransform

FileHttp -up-|> IFile
FileHdfs -up-|> IFile
FileUpyunStore -up-|> IFile

@enduml
```

