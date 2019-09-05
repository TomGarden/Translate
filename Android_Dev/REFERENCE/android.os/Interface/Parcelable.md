## Parcelable

为了类的实例可以写入 Parcel 可以实现这一接口。
实现此接口的类还必须有一个实现 Parcelable.Creator 接口的非空的静态属性 CREATOR 。