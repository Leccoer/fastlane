
# fastlane_version "2.108.0"

default_platform(:ios)

platform :ios do

  before_all do
    ENV["certificate"] = "iPhone Distribution: xxxx Co., Ltd (37WBFPMRW3)"
    ENV["mobileprovision"] = "./fastlane/xxx.mobileprovision"
    ENV["certificatePath"] = "./fastlane/xxx.p12"
  end

##key  mobileprovision  signingCertificate  configuration  certificatePath
  private_lane :xdbuild do |options|
    import_certificate(
      certificate_path: ENV["certificatePath"],
      certificate_password: "",
      keychain_password:"xxx",
      keychain_name:"login",
    )

#更新工程文件的描述文件设置
    update_project_provisioning(
      xcodeproj: "../xxx.xcodeproj",
      profile: options[:mobileprovision],
      target_filter: "RomanCar",
      build_configuration: options[:configuration]
    )

#更新teamID
    update_project_team(
      path: "../xxx.xcodeproj",
      teamid: "37WBFPMRW3"
    )

##编译并且打包
    gym(workspace: "../xxx.xcworkspace",
      scheme: "RomanCar",
      export_xcargs: "-allowProvisioningUpdates",
      output_name: "RomanCar",
      silent: true,
      clean: true,
      codesigning_identity:options[:certificate],
      configuration: options[:configuration],
      buildlog_path: "./fastlanelog",
      output_directory: "../../" ,
      export_options:{
      provisioningProfiles: {
        "com.romacredit" => "xxx"
        },
        # signingCertificate:options[:certificate],
        method: "ad-hoc",
        teamID:"37WBFPMRW3",
      }
    )
  end


  #release
  desc "Deploy a new version to the App Store"
  lane :release do
    xdbuild(
      mobileprovision:ENV["mobileprovision"],
      certificate:ENV["certificate"],
      configuration:"Release"
    )
  end
  
#debug
  lane :develop do
      xdbuild(
        mobileprovision:ENV["mobileprovision"],
        certificate:ENV["certificate"],
        configuration:"Debug"
      )
  end


  desc "上传到蒲公英"
  lane :beta do
    build_app(workspace: "xxx.xcworkspace", scheme: "xxx")
    pgyer(api_key: "xxx", user_key: "xxx")
  end

  after_all do |lane|
    # This block is called, only if the executed lane was successful

    # slack(
    #   message: "Successfully deployed new App Update."
    # )
  end

  error do |lane, exception|
    # slack(
    #   message: exception.message,
    #   success: false
    # )
  end

end
